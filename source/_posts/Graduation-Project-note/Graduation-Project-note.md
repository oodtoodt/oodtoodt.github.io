---
title: Graduation-Project-note
date: 2020-04-11 08:44:18
tags:
- home
- GP

categories:
- 
- project_notes

---

## 可用的技术
目标跟踪
目标识别
关系识别

最后这点最为重要，所以我们所有的跟踪或者识别技术都是为我们关系识别服务的——需要什么样的接口完全依赖于我们的需要罢了。
所以我们来看关系识别

### Referring Relationships
重新看了一遍，这篇老板推的关系推断对我并没有什么帮助。他的本质是把NLP或者说模式识别跟目标识别联系在一起，如果需要一个名次去指代的话，那就是visual grounding：它需要机器在接受一张图片和一个 query（指令）之后，「指」出图片当中与这个 query 所相关的物体。
很有趣的一点是这里用到了心理学：「受到心理学中移动聚光灯理论（the moving spotlight theory）的启发，通过使用谓词作为从一个实体到另一个实体的视觉注意转移操作来绕过这一挑战。」

有一点点稍微的借鉴意义；
如果我们已经固定了整个query，那么算法就退化为某种识别了。所以不是没有一点意义。

### Relation Extraction
信息抽取在自然语言处理中是一个很重要的工作，特别在当今信息爆炸的背景下，显得格外的生重要。从海量的非结构外的文本中抽取出有用的信息，并结构化成下游工作可用的格式，这是信息抽取的存在意义。信息抽取又可分为实体抽取或称命名实体识别，关系抽取以及事件抽取等。

### Visual Reasoning
Visual reasoning是个非常重要的问题，由于ResNet等大杀器出现，visual recognition任务本身快要被解决，所以计算机视觉的研究方向逐渐往认知过程的更上游走，即逻辑推理。

#### 时间关系推理（Temporal relational reasoning）
是指理解物体／实体在时间域的变化关系的能力。受启发于Relation Network，本文提出了Temporal Relation Network（TRN），用于学习和推理视频帧之间的时间依赖关系。relational resaoning一直是近期研究的热点，从图1中我们可以看出对于视频来说时序关系是重要的。

##### TSN （ Temproal Segment Networks）

视频里面的连续帧是存在很多冗余信息的，所以dense temporal sampling是不必要的，sparse temporal sampling比较合适。所以TSN的思想之一就是从长的视频中稀疏采样一些帧，然后再聚合起来，这样就能建模长时间域了。另外一个思想，TSN借鉴于two-stream的结构来同时建模appearance和dynamic。

##### Relation Network

#### 总结
我们需要了解visual reasoning的问题
>视觉推理最重要的一点就是状态的转移，这种转移往往是通过visual object之间的relation的形式，eg，被人握住的瓶子，那条狗边上的人。但现在Visual Relationship最大的问题有三：（1）现有VG数据库标记太差（2）relationship的种类数量太庞大，不像单纯的物体分类，relationship几乎很难通过标记穷尽一张图上的所有relation，理论上所有object pair有relation，尤其是空间位置关系，任何同一图上一对物体之间必定存在。尝试unsupervised方式？（3）multi-label，两两物体之间的relation不唯一，人可以同时牵着狗/在狗旁边/看着狗/。。。现在的visual relationship/scene graph框架主流还是单分类。

>作者：汤凯华
>链接：https://zhuanlan.zhihu.com/p/60418025
>来源：知乎
>著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。