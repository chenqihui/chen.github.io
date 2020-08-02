---
title: UML 建模中常用图的汇总
date: 2018-07-05 15:21:23
tags: 
  - UML
categories:
  - 软件工程
  
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这篇之前是直接放在 Github 的 [chenqihui/QHUMLModelDiagram: UML建模](https://github.com/chenqihui/QHUMLModelDiagram)，因为那时还没使用博客，如今将它迁移过来。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;而为什么整理 UML 建模呢？里面很多内容是摘录《软件工程导论（第5版）》，是读书时候的课本，现在新教材不知有没有哪些变动了。所以有出入可以联系修改，感谢。说回为什么，其实很简单，就是由于画图时常常忘了或者把多种图，在使用不熟下，交叉的使用，从而导致画出来的 UML 变成了四不像。UML 其实对开发来说，提供简单的流程、结构说，对后续优化、再次开发都很有帮助，因此还是需要认真对待。把 UML 画得不仅是给自己看，也要拿得出手给其他人查看，并且是好了解的。接下来就介绍开发中常用 UML 建模的标准定义。

<!-- more -->

#### 一、UseCase Diagrams（用例图）

>一般的用例图包含的模型元素有系统、行为者、用例及用例之间的关系。--《软件工程导论（第5版）》P224

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用例之间的关系  
1、扩展关系：同一个用例中添加一些动作后构成了另一个用例，这两个用例之间的关系就是扩展关系，后者继承前者的一些行为，通常把后者称为扩展用例。  
2、包含关系：包含关系描述的是一个用例需要某种功能，而该功能被另外一个用例定义，那么在用例的执行过程中，就可以调用已经定义好的用例。

>总结  
1.包含关系  
a.如果两个以上用例有大量一致的功能，则可以将这个功能分解到另一个用例中，其他用例可以和这个用例建立包含关系（如之前介绍的饮料自动售货机）。  
b.一个用例的功能太多时，可以使用包含关系建立若干个更小的用例。（如学生管理系统的用例图）  
2.扩展关系  
对扩展用例的限制规则：将一些常规的动作放在一个基本用例中，将可选的或只在特定条件下才执行的动作放在它的扩展用例中。

![image](https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/UseCaseDiagram1.png)


#### 二、Class Diagrams（类图）

>类图描述类及类之间的静态关系。类图是一种静态模型，它是创建其他 UML 的基础。
UML中类的图形符为长方形，用两条横线把长方形分成上、中、下3个区域（下面两个区域可省略），3个区域分别放类的名字，属性和服务。--《软件工程导论（第5版）》P217
>
>类与类之间通常有关联、泛化（继承）、依赖和细化等4种关系

##### 1、关联

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;是一种拥有的关系，它使一个类知道另一个类的属性和方法

1.1、普通关联  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;只要在类与类之间存在连接关系就可以使用，使用直线连接两个类。通常，关联是双向的，可在一个方向上为关联起一个名字，在另一个方向上起另一个名字（也可不起）。为避免混淆，在名字前面（或后面）加一个表示关联方向的黑三角。

1.2、聚集

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;也称为聚合，是关联的特例，表示类与类之间的关系是整体与部分的关系，且部分可以离开整体而单独存在。如车和轮胎是整体和部分的关系，轮胎离开车仍然可以存在。

*聚合关系是关联关系的一种，是强的关联关系；*关联和聚合在语法上无法区分，必须考察具体的逻辑关系。

1.2.1、共享聚集

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在聚集关系中处于部分方的对象可同时参与多个整体方对象的构成。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一般聚集和共享聚集使用空心菱形。

1.2.2、组合聚集，也就是1.3的组合

1.3、组合

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;是整体与部分的关系，但部分不能离开整体而单独存在。如公司和部门是整体和部分的关系，没有公司就不存在部门。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;组合关系是关联关系的一种，是比聚合关系还要强的关系，它要求普通的聚合关系中代表整体的对象负责代表部分的对象的生命周期。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用实心菱形表示。

![image](https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/ClassDiagram1.png)

##### 2、泛化

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;也叫继承，表示一般与特殊的关系，它指定了子类如何继承父类的所有特征和行为。它是通用元素和具体元素之间的一种分类关系。具体元素完全拥有通用元素的信息，并且还可以附加一些其他信息。

2.1、普通泛化

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*没有具体对象的类称为抽象类*

2.2、受限泛化

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;附加约束条件，以进一步说明该泛化关系的使用方法或扩充方法。预定义的约束有4种：多重、不相交、完全和不完全。这些约束都是语义约束。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{多重}、{不相交}、{完全}和{不完全}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不相交继承就是更多重继承相反，即一个子类不能多次继承同一个基类（这样的基类相当于C++语音中的虚基类）

##### 3、依赖

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;是一种使用的关系，即一个类的实现需要另一个类的协助，所以要尽量不使用双向的互相依赖.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用实心箭头的虚线。 
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*多表示局部变量, 方法中的参数, 和对静态方法的调用，而关联则多用于成员变量，全局变量。*

##### 4、细化

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当对同一个事物在不同抽象层次上的描述时，这些描述之间具有细化关系。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用空心箭头的虚线表示。来着P223。而在参考里面有个实现，其也使用空心箭头的虚线表示。它是一种类与接口的关系，表示类是接口所有特征和行为的实现。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;常见的有以下几种关系: 泛化（Generalization）,  实现（Realization），关联（Association)，聚合（Aggregation），组合(Composition)，依赖(Dependency)。  
各种关系的强弱顺序： 泛化 = 实现 > 组合 > 聚合 > 关联 > 依赖


#### 三、Object Diagrams（对象图）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;描述的是参与交互的各个对象在交互过程中某一时刻的状态。对象图可以被看作是类图在某一时刻的实例。


#### 四、Statechart Diagrams（状态图）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过描述系统的状态及引起系统状态转换的事件，来表示系统的行为。此外，它还指明了作为特定事件的结果系统将做哪些动作（例如，处理数据）。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;是一种由状态、变迁、事件和活动组成的状态机，用来描述类的对象所有可能的状态以及时间发生时状态的转移条件。

1、状态  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;主要有：初态（即初始状态）、终态（即最终状态）和中间状态。一个状态图只能有一个初态，而终态则可以有0至多个。  
2、事件  
3、符合  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;初态用实心圆表示，终态用一对同心圆（内圆为实心圆）表示。

![image](https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/StatechartDiagram1.png)


#### 五、Activity Diagrams（活动图）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从用户的角度描述用例

![image](https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/ActivityDiagram1.png)

#### 六、Sequence Diagrams（序列图-时序图）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从对象的角度描述用例，强调交互的时间次序。描述了对象之间传递消息的时间顺序，它用来表示用例的行为顺序。

##### 1、生命线lifeline  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;表示对象的生存时间。生命线从对象创建开始到对象销毁时终止。对象在生命线上的两种状态：休眠状态、激活状态。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;激活的符号，激活用一个细长的矩阵框（在生命线上）表示。

##### 2、消息  
2.1、同步消息  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;消息的发送者把进程控制传递给消息的接收者，然后暂停活动，等待消息接收者的回应消息。
  
2.2、异步消息  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;消息的发送者将消息发送给消息的接受者后，不用等待回应的消息，即可开始另一个活动。  

![image](https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/SequenceDiagram1.png)

#### 七、Collaboration Diagrams（协作图）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从对象的角度描述用例，强调交互的空间结构


#### 八、Component Diagrams（构件图）



#### 九、Deployment Diagrams（部署图）


#### 十、其他

##### 1、层次方框图
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;树状结构描述信息  

##### 2、Warnier图  

##### 3、IPO图
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;输入、处理、输出图的简称，它是由美国IBM公司发展完善起来的一种图形工具，能够方便地描绘输入数据、对数据的处理和输出数据之间的关系。

##### 4、包图



---

#### 参考

* [UML各种图总结-精华](https://www.cnblogs.com/jiangds/p/6596595.html)
* [UML建模](http://www.cnblogs.com/wolf-sun/category/531196.html)
* [UML关系(泛化,实现,依赖,关联(聚合,组合))](http://justsee.iteye.com/blog/808799)
* [关于startUML中各种连线这间的关系](http://blog.csdn.net/qq383264679/article/details/47832099)

#### 学习

| 单词 | 音标 | 含义 |  
| --- | --- | --- |  
| Diagram | ['daɪə,græm] | n. 图表；[数] 图解（diagram的复数）；略图<br>v. 用图解释；作…的图解（diagram的三单形式） |  
| abstract | ['æbstrækt] | n. 摘要；抽象；抽象的概念<br>adj. 抽象的；深奥的<br>vt. 摘要；提取；使……抽象化；转移(注意力、兴趣等)；使心不在焉<br>vi. 做摘要；写梗概 |  
| Association | [[əˌsəʊsɪˈeɪʃn]] | n. 协会，联盟，社团；联合；联想 |  
| Aggregation | [,ægrɪ'geʃən] | n. [地质][数] 聚合，聚集；聚集体，集合体 |  
| Composition | [,kɑmpə'zɪʃən] | n. 作文，作曲；[材] 构成；合成物 |  
| Generalization | [,dʒɛnrələ'zeʃən] | n. 概括；普遍化；一般化 |  
| Dependency | [dɪ'pɛndənsi] | n. 属国；从属；从属物 |  
| Realization | [,riələ'zeʃən] | n. 实现；领悟 |  
| Statechart |  | 状态图 |  
| Sequence | ['sikwəns] | n. [数][计] 序列；顺序；续发事件<br>vt. 按顺序排好 |  
| Collaboration | [kə,læbə'reʃən] | n. 合作；勾结；通敌 |  
| Component | [kəm'ponənt] | adj. 组成的，构成的<br>n. 成分；组件；[电子] 元件 |  
| Deployment | [dɪ'plɔɪmənt] | n. 调度，部署 |  

<!--|  |  |  |  -->  



