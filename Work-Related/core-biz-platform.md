---
title:  业务中台的演进之路 (文旅项目)
...

从一个软件工程师的角度去聊聊业务中台，努力做到通俗易懂。 

> 再装逼的名字和文案， 最后都是通过一行一行朴实的代码落地的， 业务中台在技术上和其他微服务架构的系统没有什么区别。

### 提前定义

在分享之前，我们先来学习下什么是业务中台， 方便后面的阐述。 

中台没有一个标准定义，维基百科和百度百科上是没有给这个定义的。 中台的概念是阿里巴巴在 2015 年底提出的， 相关故事就不在这里赘述。  

我们这篇文章提到的业务中台，是狭义的业务中台， 和数据中台取分开来讲。

> 有大佬的观点说 “所有的中台都是业务中台”，广义上来讲就是“所有的中台(业务中台、数据中台、技术中台等)都是为业务服务的， 为企业更快的响应市场变化，从而换取更多的利润”， 所以都是”业务中台“。  


给个我个人的程序猿式的简单定义：

**业务中台** - 位于分层架构的中间层，为上层（前台）提供可复用的业务服务的一批基础业务服务群。

> 网上有些讨论认为中台、前台还包括了组织，这里我们不做太多的争论，大家还是从具体业务价值上来看待这个事情。

中台的初衷还是提高效能，即提高企业快速响应市场变化的能力；线上系统达到这个目的的最好的手段就是”复用”。 谈到复用，程序猿出生的都很好理解，复用也是我们在代码层面重点考虑的一点，而业务中台无非是更高层面，业务层面的复用。 

那么实现业务中台最主要的就是提取可复用的基础业务；剩下的技术实现部分就是大家耳熟能详的微服务架构了。 

### 中台真的需要吗

任何时候搞一件事情，只有洞察到到其真实意义才有动力去干。 如果是指玩概念，那就是伪需求，干起来也不得劲。 

> 其实中台也是国内互联网发展内卷化带来的产物。 大家都知道国内互联网习惯性在应用层搞大量的”创新“，有个中台这样的底座，在应用层多搞些花样就方便多了  


那中台有意义吗？ 这里我们就用一个文旅项目中的门票业务的心路历程来看看中台是否有其存在的意义。

> 一个文旅平台一般包括了酒店、餐饮、门票、出行等几大业务，本文拿门票业务来作为案例，其他业务模式和门票比较类似。


### 初期架构

简单列下初期业主方的需求点：

* 搭建景区系统（存在多个景区），景区系统属于最终端的门票供应商
* 搭建小程序平台，供游客在线订票


![文旅平台中台演进 - 初期.jpg](http://tech.icoding.tech/Work-Related/文旅平台中台演进 - 初期.jpg)



### 业务升级

完成初期上线后，收到下一个迭代需求，一个月后上线， 需求如下：

* 通过平台将门票分销只美团平台上
* 部署清分系统，根据订单销售情况完成和景区之间的清分对账


 分析了美团的分销模式后，发现相对于自建平台只有支付后的业务，少了付款、退款等相关功能，而小程序是一套完整的订单业务。 同时现有模型还无法匹配支持到美团的交易接口。 这里面临两个选择：
1.  改造现有订单服务，同时支持美团分销和自建平台
2.  单独建立一个服务，独立的数据库来实现美团的分销

第一个方案风险大，可能影响现有业务，对现有代码入侵太大，于是选择了第二个方案。

![文旅平台中台演进 - 分销.jpg](http://tech.icoding.tech/Work-Related/文旅平台中台演进 - 分销.jpg)

上面的方案，可以很明显看到，在小程序门票订单和美团分销订单的服务中都涉及到订单的履行， 即：出票过程。 同时清分系统需要对接两套订单系统。这个架构其实是一个种妥协， 后期业务持续发展升级后，其弊端就逐渐显现出来。

> 门票业务订单的履行和电商订单的履行类似， 出票类比电商的发货， 退票退款类比电商的退货退款。

### 引入业务中台

在上面那个阶段，还感觉不到特别的痛； 但是接下来的需求就让人坐立不安了。 随着平台的发展，类似美团的模式已经逐渐被提出。这个时候展望平台未来的发展，仅仅是景区门票这一项业务就面临超过10个分销商和供应商。

门票这一业务模型可以参考如下图所示:

![景区门票的业务模型.png](http://tech.icoding.tech/Work-Related/景区门票的业务模型.png)

此刻很明显的感觉到，业务升级后，目前的架构已经无法适应业务快速扩展的步伐； 脑袋里面快速闪现的就是”复用“两个字。 

> ”中台“只是个更有逼格的术语而已，最终是通过复用基础业务来达到快速响应业务的扩张才是最终目的。 

这个时候你会去和业主谈复用吗？ 相信你不会， 这个时候就需要搬出”业务中台“这样有点名头的方案去争取资源，包括时间上的资源。 


### 业务中台落地

#### 定义中台业务

大的方向确定后， 耽误之际是尽快整理并定义出中台的业务边界， 整理一套中台的业务服务群从而确定方案。 这里以分销端侧来展开分享，供应商端基本类似。 

在调研市面上几大分销平台后，我们发现其接口相似度达到了80%以上， 那么中台的分销业务就需要支持120%来保证业务覆盖面。 这里不能求同去异，只能取其全集。 业务中台的第一个版本需要搞定80%的部分，剩下的在逐渐的迭代过程中来增量添加。

如果将自建平台作为一个独特的分销平台，那么一个演进的新型的架构就如同下图所示：
![统一分销平台.png](http://tech.icoding.tech/Work-Related/统一分销平台.png)


结合整体业务的分析，最终确定了中台业务包括： 门票订单中心、酒店订单中心、餐饮订单中心、产品中心、供应商中心、支付中心、清分

#### 确定业务功能

这里以分销为例聊下如何确定中台具体业务的功能范围。 

各大OTA平台的交易模型，可以通过其接口窥探一二， 根据其业务模型及交易接口来确定最终中台业务的模式和功能范围。 



确定完范围后，就需要整理出一套全量的业务接口， 类似下图所示

![分销接口.png](http://tech.icoding.tech/Work-Related/分销接口.png)

> 确定业务功能范围是个比较繁琐而必要的工作，在本文这个项目中，携程的订单模式中支持了类似购物车的概念，而其他OTA的一个订单都关联着一个唯一的产品; 具体选择哪种模式最终还需要结合业务端的规划来定。

#### 最终落地的中台架构

最终落地的业务中台的逻辑架构可以参考下图：

![逻辑架构.png](http://tech.icoding.tech/Work-Related/逻辑架构.png)



### 总结

业务中台其本身没有任何全新的概念，也不需要特殊的技术实现。 只是强化了在业务层面的复用，通过提炼沉淀可复用的业务中心，从而提高快速高效的市场响应速度，赋能应用层的快速创新。 

我们来最后总结下上面门票的业务中台带来的好处：

* 统一分销平台 实现数据的集中存储；清分对接和报表只需要从一个订单库拉取数据即可
* 统一了订单的属性，特别是订单的状态管理； 这个减轻了清分对接和后期数据报表及对账一半以上的工作量，它们处理数据时，只需要解析一个订单数据模型就可以了。
* 同时订单中心自己也只需要和供应商端，产品中心集成对接一次； 
* 整个订单的履行都在统一在一个逻辑里面，订单的生命周期管理及各种补偿机制都在一个服务中完成了复用；
* 后期分销端的扩展，只需要在前台增加一个对应的适配服务即可完成。 
* 通过在应用层添加一个开放API的网关服务即可讲能力开放出去，供比较小的OTA平台来主动对接
