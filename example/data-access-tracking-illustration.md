# 埋点说明

### 1. 概述

有新的业务项目接入数果智能平台时，或者已接入的系统升级时，需进行埋点需求定义，制定埋点计划、确定哪些地方需要埋点、是采用代码埋点还是可视化埋点、埋点的事件名称和事件类型等，形成埋点需求文档，提供给相关人员。

埋点需求人员需具备以下背景知识：

* 了解数据分析的方法，明白埋点有什么作用、是否具有分析意义；
* 了解从埋点到数据分析的整体流程及系统架构，知道自己所做的事情在整个流程中处于哪个位置，起到什么作用；
* 了解数据上报的维度定义，知道哪些数据是通过何种维度上报；
* 深入了解业务项目（网站、APP）的业务和实现逻辑，能区分是采用代码埋点还是可视化埋点，并且在埋点定义时避免漏埋或错埋；

### 2. 埋点流程

##### 2.1 流程图

![](/assets/example/data-access-tracking-illustration-flow.png)

流程说明：
* 发布项目原型：设计人员完成原型设计后，将原型发布给数据分析人员；
* 了解业务项目需求：与设计人员沟通，了解项目业务需求；
* 制定埋点计划：结合埋点原理，进行埋点定义、埋点实施、数据清洗、埋点测试、埋点部署等工作的资源安排定制；
* 埋点需求定义（可视化&代码埋点）：结合业务，定义埋点位置、埋点上报维度、埋点方式（使用可视化埋点还是代码埋点）；
* 埋点需求审核：审核埋点需求，保证埋点准确性与合理性；
* 发布可视化埋点需求：将代码埋点发布给开发相关人员；
* 发布代码埋点需求：将可视化埋点发布给埋点执行人员；

##### 2.2 发布项目原型

项目产品设计完成后，就可将项目原型发布给数据分析人员了，在原型正在开发的过程中，同步分析项目业务需求，及时沟通，能起到快速调整项目与业务相关功能。

##### 2.3 了解业务项目需求

数据分析人员与项目产品设计人员了解项目的业务与需求，以项目业务原型为基础开展数据分析计划。在开展工作时，可注意以下几点：
* 在项目业务原型发布到完成的过程中，原型、功能、业务可能会不断地修改。相关人员在对应负责领域进行修改后，需通知相关人员进行沟通调整；
* 项目业务的实现与原型有差异时，确定以哪个作为标准，并可开启测试环境进行功能对比；
* 项目业务原型需及时根据调整修改原型设计，并与业务数据分析人员沟通；

##### 2.4 制定埋点计划

埋点计划包括埋点定义、埋点实施、数据清洗、埋点测试、埋点部署等工作的时间与人员安排。

备注：埋点计划中，埋点部署与项目正式上线发布的日期最好尽量同步。

##### 2.5 埋点需求定义

埋点需求定义主要针对页面浏览、按钮点击、元素对焦这3类事件，确定事件的页面名称、事件名称、事件类型。

埋点需求人员根据埋点定义规则确定项目业务哪些地方需要埋点，是采用可视化埋点还是代码埋点，并定义页面名称、事件名称等，形成相应的埋点文档。

比较推荐的一些命名规则如下：

###### 2.5.1 页面名称

* 根据项目业务的操作系统区分（如PC端网页、Android、iOS），对于业务相同的页面，可以使用同名的页面名称，当需要区分来源时，可以查看数据的`system_name`维度进行区分。
* 对于同一项目中的同一应用，页面名称需唯一，从而区分页面，以便数据分析。
* 对于不同应用的业务相同页面，页面名称相同能统一统计页面数据。
* 清晰易懂的页面名称能使数据分析事半功倍。
* 在无重名的页面情况下，可直接以导航栏的标题名称作为页面名称，若出现重名的页面，可以使用“场景”+“标题”或“功能”+“标题”作为页面名称，若页面本身没有导航标题，则可以页面功能作为页面名称。

###### 2.5.2 事件名称

* 根据项目业务的操作系统区分（如PC端网页、Android、iOS），对于业务相同的事件，可以使用同名的事件名称，当需要区分来源时，可以查看数据的`system_name`维度进行区分。
* 对于同一项目中的同一应用，事件名称需唯一，从而区分事件，以便数据分析。
* 对于不同应用的业务相同事件，事件名称相同能统一统计事件数据。
* 清晰易懂的事件名称能使数据分析事半功倍。
* 事件名称与页面名称是不同的分析维度，是允许同名的。
* 对于跨页面的相同按钮，可使用相同的事件名称。
* 若按钮有名称，不存在重名事件，则可以按钮名称作为事件名称。若按钮有名称，存在同名事件，可以使用场景+按钮名称”或者“功能+按钮名称”作为事件名称。同类按钮或者没有名称的按钮，根据按钮功能定义事件名称，事件名称尽量唯一。
* 通用类按钮，如“确定”、“取消”，加上功能名称能使数据统计时更加直观。

###### 2.5.3 可视化埋点 & 代码埋点

埋点按实施对象划分，可分为两种：代码埋点和可视化埋点。

通过原型判断是代码埋点还是可视化埋点：

* 页面路径（URL）发生变化： 可视化埋点根据URL标识一个唯一页面，URL变化则认为是不同的页面。因此，如果一个页面对应的URL不确定，则无法可视化埋点。
* 一个按钮多种状态： 可视化埋点根据元素路径标识一个唯一按钮，如果同一个元素路径有多种状态，则无法可视化埋点。
* 系统菜单，多个页面存在： 多个页面存在的系统菜单，每个页面都要进行埋点，如果有几十个或上百个页面，可视化埋点需埋点多次，建议通过代码埋点实现。

在实施埋点过程中，可能会发现其他无法可视化埋点需改为代码埋点的情况，需及时与相关人员沟通。


##### 2.6 埋点需求审核

埋点需求审核可组织埋点小组成员和业务部门形成审核小组，对埋点需求进行审核。审核的重点在于埋点的准确性和合理性。规则如下：
* 埋点内容无遗漏，与原型相符；
* 页面名称和事件名称的定义符合埋点规则；
* 数据类型的定义符合数据类型命名规则；
* 埋点需求文档明确区分代码埋点、可视化埋点；
* 埋点具有分析价值。

##### 2.7 发布可视化埋点需求

将定义好的可视化埋点需求发布给相应的业务人员。

##### 2.8 发布代码埋点需求

将定义好的代码埋点需求发布给相应的项目开发者。
