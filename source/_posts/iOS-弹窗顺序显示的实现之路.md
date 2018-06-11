---
title: iOS 弹窗顺序显示的实现之路
date: 2018-06-11 11:39:46
tags:
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当产品搞了一堆弹窗的需要后，如各自提示、引导，然后发现这样不好，需要进行排序显示，也就是显示一个之后再显示下一个，这事就落到开发手上。这些弹窗有各种不同类型，并且每种弹窗可能有取消（好的），或者还有确定（来进入下一弹窗)，也就是说整个业务完成之后才算消失。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每个弹窗显示的内容是根据服务端下发的数据来配置。下发的方式有 HTTP （一次性下发所有需要显示的弹窗数据）或 Socket （分开下发各个弹窗数据，也就是说下发的先后顺序可能不同）。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;场景描述完了，用我的真实需求来说明：弹窗分别有心愿审核结果（A）、心愿填写（B）、心愿分享引导（C）。A有两种状态，审核通过（A1，只有确定，点击则消失）和审核拒绝（A2，有取消和确定，取消点击则消失，确定则进入下个流程），在 A2 时点击确定则显示 B，然后可以进行心愿的修改，也可以取消来结束，当上面流程结束后显示 C，以上都需要先判断是否有 A、B、C 可供显示，有就显示，无则跳过。下面是简单的顺序图，任意节点没有则跳过，直接到下一个。

				A
			/		\
		  A1		A2
		  	\		  \
		  	C		   B
					  /
					 C 

<!-- more -->

接着聊聊排序显示的实现：

#### 一、

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最初的解决方案是想简单暴力点，因为细看上面的需求，其实就是两层。要记录的就是 A 是否出现，是否点击显示 B，然后是否显示显示 C 。前面说的如果是 Socket 下发 A 和 C 时，暂不考虑 C 在 A 前面的情况（其实后续实现 FIFO 就可以不理会），因为目前需求是 A 肯定在 C 前面。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过 bShowWishResult 记录是否显示 A，而接着是显示 B 则等 B 消失时把 bShowWishResult = NO，不然则 A 消失时置 NO，此时再通过 shareGuideDic 判断是否显示 C。当 A 和 C 同时下发，C 先判断 bShowWishResult，为 NO 才显示，不然先记录 shareGuideDic 再等待。代码实现如下：

~~~Objective-C
// 是否显示心愿结果 A 中
@property (nonatomic) BOOL bShowWishResult;
// 心愿分享引导 C 的数据
@property (nonatomic, strong) NSDictionary *shareGuideDic;

- (void)doConnectSocket:(NSDictionary *)dic {
	if (type == A) {
		self.bShowWishResult = YES;
		[self p_showWishResultView:dic];
	}
	else if (type == C) {
		if (self.bShowWishResult == YES) {
			self.shareGuideDic = dic;
		}
		else {
			[self p_showShareGuideView:dic];
		}
	}
}

// 显示 A 页面
- (void)p_showWishResultView:(NSDictionary *)dic {
	...
	// 显示完消失，通过 delegate 或者 block 来调用 p_nextView 显示 C
	self.bShowWishResult = NO;
	[self p_nextView];
}

// 显示 C 页面
- (void)p_showShareGuideView:(NSDictionary *)dic {
	// 显示完消失，通过 delegate 或者 block 来调用，来清空记录
	self.shareGuideDic = nil;
}

- (void)p_nextView {
	// 有 C 的数据则显示 C 页面
	if (self.shareGuideDic != nil) {
		[self p_showShareGuideView:self.shareGuideDic];
	}
}
~~~

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不论是 A 之后直接到 C， 还是 A 到 B，再到 C，都得通过 delegate 或者 block 到主 VC 来调用 ***[self p_nextView];*** 来显示下个弹窗。这里有个问题是每个页面的都需要这种消失的回调。而类似 A1，本来就是个简单的提醒弹窗，此时就不得不也添加回调。这样等于增加了总的代码量，主 VC 也增加处理。所以后面会讨论是否使用 ***监听*** 来减低这种耦合。

#### 二、

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当完成第一版的时候，已经满足了需求。可是再细想下，会发现其实这样做并不通用，也就是说再增加一个类似A或者C的弹窗时候，需要再增加 property 的字典等逻辑。而这种排序显示让人想到了堆（heap），FIFO。所以使用堆的数据结构来保存这种顺序的话，然后再拿出来显示。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;需要增加：项目中 Socket 过来的数据是要判断同一场次的，就是推 type 为 A 的gameId 需要和 type 为 B 的 gameId 要一致。gameId 是在进场的 HTTP 或者初始的 Socket 路由得到的，我们额外保存当前 gameId 就可以。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;现在我们使用 heap 管理类，将同个 gameId 的弹窗数据一一添加到对应 key-value 数组中。这样就不用因为多个弹窗再添加类似第一版的 bShowWishResult 和 shareGuideDic，现在直接通过 QHHeapManager 的 queueDicValue 来管理。代码如下：

~~~
#define QHHeapTypeKey @"type"
#define QHHeapDicKey @"dic"

typedef enum : NSUInteger {
    QHHeapTypeWish,
    QHHeapTypeShare
} QHHeapType;

@interface QHHeapManager()

@property (nonatomic, strong) NSMutableDictionary *queueDicValue;

@end

@implementation QHHeapManager

- (NSArray *)hasNext:(NSSting *)gameId {
    NSArray *value = self.queueDicValue[gameId];
    if (value != nil && value.count > 0) {
        return [value copy];
    }
    return nil;
}

- (void)addQueue:(NSSting *)gameId value:(nonnull id)data {
    NSArray *value = self.queueDicValue[gameId];
    if (value != nil) {
        NSMutableArray *newValue = [NSMutableArray arrayWithArray:value];
        [newValue addObject:[data copy]];
        [self.queueDicValue setObject:newValue forKey:gameId];
    }
    else {
        [self.queueDicValue setObject:@[[data copy]] forKey:gameId];
    }
}

- (void)goNext:(NSSting *)gameId {
    NSArray *value = self.queueDicValue[gameId];
    if (value != nil && value.count > 0) {
        NSMutableArray *newValue = [NSMutableArray arrayWithArray:value];
        [newValue removeObjectAtIndex:0];
        [self.queueDicValue setObject:newValue forKey:gameId];
        if (newValue.count > 0) {
        	// 下一个
       		[self.delegate nextQueue:[newValue firstObject]];
        }
    }
}

@end
~~~

调用方式，及回调逻辑代码：

~~~
// A
[self.heapManager addQueue:@"123" value:@{QHHeapTypeKey: @QHHeapTypeWish, QHHeapDicKey: dic}];
// B
[self.heapManager addQueue:@"123" value:@{QHHeapTypeKey: @QHHeapTypeShare, QHHeapDicKey: dic}];
// 完成 A 之后，调用
[self.heapManager goNext:@"123"];

// delegate
- (void)nextQueue:(id)value {
	NSDictionary *dic = ((NSDictionary *) alue);
	if ([dic[QHHeapTypeKey] integerValue] == QHHeapTypeShare) {
		[self p_showShareGuideView:dic[QHHeapDicKey]];
	}
}
~~~

#### 三、

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;说到这里可以看出 QHHeapManager 的 API 只支持用于 gameId，或者类似 gameId 的 key。但是注意他们不能一起使用，除非使用特殊字符区分，必须唯一。而 delegate 的 ***- (void)nextQueue:(id)value;*** 也需要区分大类，再区分小类。因此需要修改上面的 delegate 的API。最终我通过增加枚举来维护不同的字段，这样也方便查看目前使用的排序种类，避免重复。而下面的完整版也增加了上面提到的 NSNotificationCenter 来解耦，让 A1 无须多写 delegate，只需在页面消失添加 Notif 的 post。代码如下：

```
[[NSNotificationCenter defaultCenter] postNotificationName:GONEXT_LUACYQUEUE_NOTIFICATIONNAME object:@(QHQueueTypeEnter)];
```
QHHeapManager 的完整代码如下：

~~~
//  通知
#define GONEXT_LUACYQUEUE_NOTIFICATIONNAME @"GONEXTLUACYQUEUENOTIFICATIONNAME"
// 增加大类
typedef enum : NSUInteger {
    QHQueueTypeFinal,
    QHQueueTypeEnter
} QHQueueType;

typedef enum : NSUInteger {
    QHHeapTypeWish,
    QHHeapTypeShare
} QHHeapType;

#define QHHeapTypeKey @"type"
#define QHHeapDicKey @"dic"
// 大类的 key
#define QHHeapOtherKey @"otherKey"

#define QHQueueValue @"queueValue"

@interface QHHeapManager()

@property (nonatomic, strong) NSMutableDictionary *queueDicValue;

@end

@implementation QHHeapManager

- (void)dealloc {
    [[NSNotificationCenter defaultCenter] removeObserver:self name:GONEXT_LUACYQUEUE_NOTIFICATIONNAME object:nil];
    _queueDicValue = nil;
}

- (instancetype)init {
    self = [super init];
    if (self) {
        [self p_setup];
    }
    return self;
}

- (void)p_setup {
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(goLuckyQueueNextAction:) name:GONEXT_LUACYQUEUE_NOTIFICATIONNAME object:nil];
}

- (NSArray *)nextValue:(QHQueueType)type {
    NSArray *value = self.queueDicValue[@(type)];
    if (value != nil && value.count > 0) {
        return [value copy];
    }
    return nil;
}

- (void)deleteValue:(QHQueueType)type {
    [self.queueDicValue removeObjectForKey:@(type)];
}

- (void)addQueue:(QHQueueType)type value:(nonnull id)data {
    NSArray *value = self.queueDicValue[@(type)];
    if (value != nil) {
        NSMutableArray *newValue = [NSMutableArray arrayWithArray:value];
        [newValue addObject:[data copy]];
        [self.queueDicValue setObject:newValue forKey:@(type)];
    }
    else {
        [self.queueDicValue setObject:@[[data copy]] forKey:@(type)];
    }
}

- (void)goNext:(QHQueueType)type dicComplete:(BOOL(^)(QHQueueType type, id value))complete {
    if (type == QHQueueTypeFinal ||
        type == QHQueueTypeEnter) {
        NSArray *value = self.queueDicValue[@(type)];
        if (value != nil && value.count > 0) {
            NSMutableArray *newValue = [NSMutableArray arrayWithArray:value];
            [newValue removeObjectAtIndex:0];
            [self.queueDicValue setObject:newValue forKey:@(type)];
            if (newValue.count > 0) {
                if (complete == nil || complete(type, [newValue firstObject]) == YES) {
                    if (newValue.count > 0) {
                        [self.delegate nextQueue:type value:[newValue firstObject]];
                    }
                }
            }
        }
    }
}

- (void)goLuckyQueueNextAction:(NSNotification *)notif {
    id type = notif.object;
    if (type != nil) {
        [self goNext:[type integerValue] dicComplete:nil];
    }
}

- (NSMutableDictionary *)queueDicValue {
    if (_queueDicValue == nil) {
        _queueDicValue = [NSMutableDictionary new];
    }
    return _queueDicValue;
}

@end


#pragma mark - QHHeapManagerDelegate

- (void)nextQueue:(QHQueueType)type value:(id)value {
    if (type == QHQueueTypeFinal) {
        NSDictionary *dicValue = value;
        if ([dicValue[QHHeapTypeKey] integerValue] == QHHeapTypeShare) {
            [self p_showFinalShareGuideView];
        }
    }
    else if (type == QHQueueTypeEnter) {
        NSDictionary *dicValue = value;
        if ([dicValue[QHHeapTypeKey] integerValue] == QHHeapTypeWish) {
            [self p_wishAction];
        }
    }
}
~~~

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意上面有个 goNext 函数的增加了 block，用于对已保存的字典数据进行再比较，来判断是否继续 goNext。如下：

~~~
- (void)goNext:(QHQueueType)type dicComplete:(BOOL(^)(QHQueueType type, id value))complete;
~~~

总结：   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;目前的解决方案是使用字典，大类作为key，内容及小类使用数组。其实就是利用数组的堆结构，add 和 firstObject，从而达到 FIFO，顺序获取需要显示的弹窗类型及数据等。而增加的通知，它虽然能减耦，但是但有多个类都创建 ***QHHeapManager*** 时，会有多处收到通知，可是通过类型判断可以避免触发对应逻辑（这也是为什么每个顺序都得添加一个大类），但是这种通知是消耗资源的。所以页面的逻辑要以使用回调为主，通知为辅。