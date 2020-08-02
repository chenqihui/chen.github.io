---
title: iOS 弹幕系统
date: 2019-01-16 11:02:57
tags: 
  - layer
  - 动画
  - GCD
categories:
  - iOS
  
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;新弹幕系统 [chenqihui/QHDanmu2Demo](https://github.com/chenqihui/QHDanmu2Demo)，它在 [iOS 飞屏的实现之路 | RyuukuSpace](http://chenqihui.github.io/2018/06/26/iOS-%E9%A3%9E%E5%B1%8F%E7%9A%84%E5%AE%9E%E7%8E%B0%E4%B9%8B%E8%B7%AF/) 里面的也提到过，与飞屏在展示与运动逻辑的实现上基本相似，区别可能在于弹幕系统会更加注重弹幕在轨道的选择，因此本次也着重介绍下弹幕系统的轨道选择部分。

<!-- more -->

#### 轨道选择

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;弹幕直白点说就是在视频上显示文本，而如何显示文本，以至于它们之间不会相互遮拦。这就需要根据情况判断弹幕的出现轨道。下面介绍已飘屏的弹幕实现（这也是大部分弹幕的效果，也有时渐隐类的弹幕等等）

<div align=center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/Danmu/dmz2.gif" width=50% height=50%></div>

##### 碰撞

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;碰撞是弹幕需要避免的计算，特别是在大量弹幕显示的时候，如何控制在不碰撞或者更低的碰撞率下显示更多的弹幕。

<!--<div align=center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/Danmu/dm1.png" width=60% height=60%></div>-->

<table>
    <tr>
        <td ><center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/Danmu/dm1.png">错误 ❌ </center></td>
        <td ><center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/Danmu/dm2.png">正确 ⭕️</center></td>
    </tr>
</table>



##### 深度优先 & 广度优先   

* 深度优先：从上往下，优先选择可满足运动的轨道  
* 广度优先：从上往下，优先选择空闲轨道（即无弹幕），没有才选择可满足运动的轨道  
* 满足可运动：新弹幕加入后，在最后弹幕运动完成过程中不存在碰撞，即新弹幕不会与最后弹幕重叠。

<table>
    <tr>
        <td ><center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/Danmu/dm3.png">深度优先 </center></td>
        <td ><center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/Danmu/dm4.png">广度优先</center></td>
    </tr>
</table>

##### 深度优先的逻辑大致为：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;运动的总时间 t，屏幕宽度 ws，而最后弹幕 d1 长度 w1 & 速度 v1（(w1+ws)/t），而新弹幕 d2 长度 w2 & 速度 v2（(w2+ws)/t）。它们分别加入的时间为s1、s2，而时间差 bt（s2-s1）。运动都是匀速运动。

* 1、当 d1 在 dt 时间内是否运行了 w1 的距离，否的话则该轨道不可使用（w1 > dt*v1），是则往下继续判断  
* 2、当 d1 在 dt 时间内已运行了 >=w1 的距离，此时看 d2 是否能在该轨道碰撞 d1
	* 2.1、d2 速度小于等于 d1（v2 <= v1），则不会碰撞，该轨道可使用  
	* 2.2、否的话，看 d2 在 ws 运行的时间是否大于 d1 运动完需要的剩余时间（ws/v2 >= (t-bt)），是的话则不会碰撞，该轨道可使用，否则不可使用  
* 3、其余其他情况都是会碰撞  


```objc
- (QHDanmuCellParam)p_danmuParamWithPlayUseTime:(CGFloat)playUseTime {
    NSDictionary *data = _danmuDataList.firstObject;
    QHDanmuCellParam newParam;
    newParam.pathwayNumber = -1;
    
    // 广度优先：先搜索是否有空闲的轨道，有则使用，无则再走深度优先
    if (_searchPathwayMode == QHDanmuViewSearchPathwayModeBreadthFirst) {
        for (int i = 0; i < _pathwayCount; i++) {
            NSIndexPath *indexPath = [NSIndexPath indexPathForRow:i inSection:0];
            id obj = [_cachedCellsParam objectForKey:indexPath];
            if ([obj isKindOfClass:[NSNull class]] == YES) {
            	// ...
                break;
            }
        }
        
        if (newParam.pathwayNumber >= 0) {
            return newParam;
        }
    }
    
    for (int i = 0; i < _pathwayCount; i++) {
        
        NSIndexPath *indexPath = [NSIndexPath indexPathForRow:i inSection:0];
        id obj = [_cachedCellsParam objectForKey:indexPath];
        if ([obj isKindOfClass:[NSNull class]] == YES) {
           // ...
        }
        else if ([obj isKindOfClass:[NSValue class]] == YES) {
            CFTimeInterval spaceTime = newParam.startTime - lastParam.startTime;
            // 1、
            if (spaceTime * lastParam.speed >= lastParam.width) {
            	   // ...
                
                // 2、最后一个弹幕已完全显示在轨道
                // 2.1、如果最后弹幕的速度大于新的弹幕速度，则该轨道可与使用
                if (lastParam.speed >= newParam.speed) {
                    newParam.pathwayNumber = lastParam.pathwayNumber;
                }
                else {
                    // 2.2、最后一个弹幕剩余的滑动时间，新弹幕是否能在显示区域追上，判断新弹幕需要的时间与之比较。小于等于，则追不上，该轨道可使用；反之不可使用。
                    CGFloat useTimeInScreen = self.frame.size.width / newParam.speed;
                    if (useTimeInScreen >= (playUseTime - spaceTime)) {
                        newParam.pathwayNumber = lastParam.pathwayNumber;
                    }
                }
                // 3、
            }
            else {
                // 3、最后一个弹幕还没有完全显示在轨道（即尾部还未显示），则该轨道不可使用
            }
        }
        
        if (newParam.pathwayNumber >= 0) {
            break;
        }
    }
    
    return newParam;
}
```

#### QHDanmuView & QHDanmuViewCell

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考 UITableView & UITableViewCell 的接口、逻辑的设计

* View 的创建 & 添加
* Cell 的注册 & 获取
* Cells 的复用
* DataSource & Delegate 实现轨道数量 & 轨道高度等的配置

```objc
// 保存已经使用的 Cells
@property (nonatomic, strong) NSMutableArray *reusableCells;
// 记录注册的 Cell class & Identifier
@property (nonatomic, strong) NSMutableDictionary *reusableCellsIdentifierDic;
// 每个轨道最后且正在运动的弹幕参数：速度、长度、轨道编号 & 起始时间。
@property (nonatomic, strong) NSMutableDictionary *cachedCellsParam;

// 注册 Cell
- (void)registerClass:(nullable Class)cellClass forCellReuseIdentifier:(nonnull NSString *)identifier;

// 获取 Cell
- (nullable __kindof QHDanmuViewCell *)dequeueReusableCellWithIdentifier:(nonnull NSString *)identifier;

// 运动前后对于数据的管理 & Cell 的复用管理
- (void)p_danmuAnimationOfFlyWithCell:(QHDanmuViewCell *)cell param:(QHDanmuCellParam)param playUseTime:(CFTimeInterval)playUseTime;
```

参考

* [Chameleon/UITableView.m](https://github.com/BigZaphod/Chameleon/blob/master/UIKit/Classes/UITableView.m)
* [UITableView的Cell复用原理和源码分析](https://www.jianshu.com/p/5b0e1ca9b673)
* [UITableviewCell复用机制](https://www.jianshu.com/p/1046c741fce1)

#### 基础操作

```
// 弹幕加入
- (void)insertData:(NSArray<NSDictionary *> *)data;
- (void)insertDataInFirst:(NSDictionary *)data;
// 清空
- (void)cleanData;

// 弹幕的开关、恢复 & 暂停
- (void)start;
- (void)stop;
- (void)resume;
- (void)pause;
```

#### Manager

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;前面是直播使用的弹幕（即加入后显示，实时显示）。但对于点播的弹幕，其会提前加载一段时间点的弹幕，然后再根据弹幕的时间戳进行显示。因此 Manager 就是用来管理这些弹幕，根据与点播视频的时间进度匹配相应的弹幕，然后加入弹幕系统的总控制。

* 新弹幕进行排序
* 加入到总弹幕池进行合并排序
* 通过播放的时间点来匹配符合（即小于等于当前时间点）的弹幕
* 批量加入弹幕系统

```
- (void)setMediaPlayAbsoluteTime:(CFAbsoluteTime)mediaPlayAbsoluteTime {
    _mediaPlayAbsoluteTime = mediaPlayAbsoluteTime;
    int index = -1;
    if (_danmuDataList.count > 0) {
        for (int i = 0; i < _danmuDataList.count; i++) {
            NSDictionary *danmuData = _danmuDataList[i];
            CFTimeInterval startTime = 0;
            // 获取时间戳
            startTime = _startTimeBlock(danmuData);
            // 与当前时间点比较
            if (startTime <= mediaPlayAbsoluteTime) {
                index = i;
            }
            else {
                break;
            }
        }
        if (index >= 0) {
            NSArray *inData = [_danmuDataList subarrayWithRange:NSMakeRange(0, index + 1)];
            [_danmuView insertData:inData];
            [_danmuDataList removeObjectsInArray:inData];
        }
    }
}
```

#### 其他

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上是弹幕系统在单一弹幕展示上基础的功能实现。项目的 dev-0.3 分支正在增加弹幕系统支持更多运动模式（如渐隐渐显）。思路是通过不同 section 来区分，类似 UITableView 的 section，不过还得思考如果有指定运动区域（如上中下）的通用性。




