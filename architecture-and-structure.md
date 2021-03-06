# 技术架构与组织结构的演变路径

技术架构和组织结构决定了研发产能的极限，提升产能的典型手段是增加人数，`增加人数 = 提升产能` 这个公式成立的关键因素是软件开发是工程师手工生产的过程，不是工业化生产的过程，本文尝试从优化技术架构和组织结构的角度探索提升研发产能的路径。

## 传统组织结构

传统模式下，当业务扩展到一定规模后通常会进行纵向划分，每个业务下有一个全能小团队，由产品经理、设计师、前后端工程师、测试工程师等各种角色组成。

```
+-------------------------------------------------------+
|                                                       |
|   产品线 A   |   产品线 B   |   产品线 C   |   产品线 D   |
|    -----    |    -----    |    -----    |    -----    |
|     PM      |     PM      |     PM      |     PM      |
|     RD      |     RD      |     RD      |     RD      |
|     QA      |     QA      |     QA      |     QA      |             
|     ..      |     ..      |     ..      |     ..      |
|                                                       |
+-------------------------------------------------------+
```

好处很明显，小团队行动迅速，支持产品的快速迭代。

产品线之间通常相互独立，交集很少。具体到研发，做得比较好的可能会有统一技术栈、工程工具、基础平台的支撑，对研发效率的提升帮助很大，但业务开发依然会强依赖产品线各个研发团队的能力。

这种模式是浅层基础设施 + 重业务团队的组合。

```
+-------------------------------------------------------+
|                                                       |
|                       重业务团队                        |
|                                                       |
|   产品线 A   |   产品线 B   |   产品线 C   |   产品线 D   |
|    -----    |    -----    |    -----    |    -----    |
|     PM      |     PM      |     PM      |     PM      |
|     RD      |     RD      |     RD      |     RD      |
|     QA      |     QA      |     QA      |     QA      |             
|     ..      |     ..      |     ..      |     ..      |
|                                                       |
+-------------------------------------------------------+
|                                                       |
|                      浅层基础设施                       |
|                                                       |
|                 基础技术栈、工程工具、平台...             |
|                                                       |
+-------------------------------------------------------+
```

瓶颈也很明显：

* 小团队的研发能力决定了产品线的迭代速度。
* 开辟新业务时团队的能力结构很难跟上，业务开发通常需要老师傅 + 小师傅的研发组合，难以快速扩张。
* 产品线人力不均衡，各个产品线在不同阶段人力的饱和或紧缺成为常态。
* 人员成长路径受限。

## 理想组织结构

解决以上问题需要一种有别于传统组织结构的模式。

```
+-------------------------------------------------------+
|                                                       |
|   产品线 A   |   产品线 B   |   产品线 C   |   产品线 D   |
|                                                       |
+-------------------------------------------------------+
|                                                       |
|                       轻业务团队                        |
|                                                       |
+-------------------------------------------------------+
|                                                       |
|                      产品组装能力                       |
|                                                       |
+-------------------------------------------------------+
|                                                       |
|                       强基础设施                        |
|                                                       |
+-------------------------------------------------------+
```

基础设施会有更多贴近业务的沉淀，技术层面强调产品的可组装，而不是纯编程实现，在基础设施和产品组装能力的支撑下，业务开发对研发的能力结构的要求会大幅降低。

基础设施和产品组装能力需要持续迭代强化，主要分两个方向：

* 工程师能力的沉淀，传统模式下每个产品线的研发团队都需要一些高阶工程师来主导，工程师的能力有很多重合：业务理解实现能力、全局观、技术视野等等，这些能力要转化为技术资产沉淀到基础设施中。
* 提炼共性、管理差异，共性和差异表达了应用开发的全部内容，这两方面的沉淀决定了产品组装能力。

## 技术架构

技术架构会影响组织结构，研发团队角色的划分和协作方式基本都来自技术架构的设定。

具体到前端，典型的应用开发模式是：

```
+-------------------------------------------------------+
|                                                       |
|    产品 A   |    产品 B    |    产品 C    |    产品 D    |
|                                                       |
+-------------------------------------------------------+
|                                                       |
|               组件库（UI 组件、业务组件）                 |
|                                                       |
+-------------------------------------------------------+
|                                                       |
|            基础技术栈（基础库、框架）、工程工具             |
|                                                       |
+-------------------------------------------------------+
```

基础技术栈提供了分层（数据层、视图层）和组件化的支持，组件化之上沉淀出通用组件库，在此之上再开发各个产品，基本贴近传统模式下按业务纵向划分的结构。

向理想组织结构演变需要对技术架构作出调整：

```
+-------------------------------------------------------+
|                                                       |
|    产品 A   |    产品 B    |    产品 C    |    产品 D    |
|                                                       |
+-------------------------------------------------------+
|                                                       |
|             业务单元库（可以独立运行的业务模块）            |
|                                                       |
+-------------------------------------------------------+
|                                                       |
|               组件库（UI 组件、业务组件）                 |
|                                                       |
+-------------------------------------------------------+
|                                                       |
|            基础技术栈（基础库、框架）、工程工具             |
|                                                       |
+-------------------------------------------------------+
```

在典型开发模式上加了一层 `业务单元库`，由此带来的关键转变是：开发公式由 `应用` = `数据层` + `视图层（组件树）` 改为 `应用` = `{业务单元集合}`。

每个业务单元承载一块完整的业务逻辑，最终产品由业务单元组装而成，不同产品的组装是类似的过程，这个过程需要实现一些通用的代码逻辑，除此之外还会有一些应用的全局逻辑沉淀到通用代码中，通用代码的产出过程就是 `把工程师能力转化为技术资产` 的典型范例。

调整后的结构会带来明显的收益：

* 业务扩张和人员调配更灵活，传统结构的组合是：`产品线` = `{老师傅 + 小师傅的集合}`，新的模式下更强调业务单元的划分，弱化了产品线的划分，新的组合是：`{产品线集合}` = `{业务模块集合}` = `{工程师集合}`。
* 工程师的价值会得到提升，过去只支持单个产品线的业务，将能力沉淀到基础设施后可以产生规模价值。
* 在技术架构的支持下，能力的沉淀会逐渐成为团队的工作习惯，基础设施和产品组装能力持续强化，团队产能会持续提升。

以上是对提升研发产能的一点探索，从手工生产到工业化生产还有很长的路要走。

> 皮成，2019.08.15