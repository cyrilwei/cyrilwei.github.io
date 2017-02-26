---
layout: post
title:  "Modern Mobile Strategy"
date:   2017/02/26 10:41:00 +0800
categories: iOS
---

##### _Note: 这里说的策略指的是技术策略，并且主要针对需要构建团队长期开发和维护app的公司。原型开发和独立app开发不需要考虑这么多。原型开发就用你最熟悉的，独立app开发就用你最喜欢的，就是这么任性。_
##### _Note: The strategy we are talking about here is the technical strategy. It's mainly for companies that need a long term mobile team to build and maintain business apps. Prototype developments or indie developers don't need to consider such things. For prototype just use the most familiar way, and for indie just use the one you like more, that's it._

### **Why mobile strategies**

基于智能手机的移动开发起源于所谓的'native'开发方式。在最开始的一段时间里，移动开发基本上就仅有native一种方式而已。
The development for smartphone started from the way called 'native'. This is the only way to develop smartphone apps at the beginning.

虽然人们很快就试图将desktop和web开发中的各种跨平台经验搬到mobile端，不过到目前为止还没有完美的解决方案。于是就出现了基于不同方案的所谓的“Mobile Strategies”。
Although people managed to bring cross platform experiences from desktop and web to mobile world. But there is no perfect solution yet. That's why we have many 'Mobile Strategies' based on different solutions.

随着各行业对app的依赖越来越强，企业所发布的app不论从功能上还是数量上都在上升，对于大公司来说，维持一个拥有足够能力的移动开发团队就变成了一个很棘手的问题。
Following the increasing demand of mobile apps in many businesses.

障碍主要来源于两个方面：
The difficult comes from two main aspects:

1. 主流的移动平台有两个：iOS和Android（确实存在一些企业希望能够支持基于Windows的移动设备）。而为这两个平台开发app可不是“不兼容”那么简单。从语言，SDK，到设计风格，系统服务的架构，基本就没有一样的。这也就意味着如果企业很注重客户体验的话，那么从设计，开发，到部署，都需要维护两个技术栈完全不同的团队，并且这两个团队还要在业务层面保持一致。因此，不管是人力成本，还是沟通成本都很可观。
1. There are two main mobile platform: iOS and Android. Developing for those two platform is not only a simple 'incompatible'. There is nothing similar in almost everything from the languages, SDKs, design styles, to the architecture of system services. That also means if the company cares about the user experiences, it needs to maintain two teams for designing, developing, and deploying with different technologies. And the two teams have to be aligned in the business level. This would be a big cost on both human resources and communications.

2. 因为移动开发相对其他平台还比较新。拥有丰富经验和技术能力的开发人员远少于市场实际需求。对于很多公司来说，拥有足够尺寸的native团队就成了一个奢望。
2. The mobile development is quite new to other platform. There is not enough experienced developers in the market. For many companies, having a big enough native team is unpractical.

基于以上原因，大多数公司都希望能够选择一个符合自己的需求和能力，并且可以在未来至少3-5年内可以放心的进行稳定投入的开发策略。（对于移动领域来说，3-5年已经算是很长了）

### **What mobile strategies we have Now**

这里我们主要从app端的层面来说一下所谓的strategies。

mobile技术策略，也叫技术选型。这个是讨论的最多的也是最多选项的strategies。在很多场合，狭义的mobile strategies仅指技术strategies。

传统strategies主要有三大分类: `Native`, `Cross Platform`, `Mobile Web`。其中的`Cross Platform`又可以细分以Microsoft的"Xamarin"和Facebook的“React Native”为代表的所谓的`Native cross platform`，和由Apache Cordova及其各种变种统领的`Hybrid cross platform`。

### **What's wrong with those strategies?**

目前我见到的最大的问题是，大多数公司在提到移动策略的时候，都是把上述几种类别进行*对立*来考虑的。比如“我们应该用Native还是Hybrid？”，“是iOS用native，其他平台用hybrid？还是其他平台用mobile web就行了？”，“同样是cross platform，我们应该用React Native还是Cordova？”之类。

在移动开发早期，各种技术还处在探索和磨合期，有这样的顾虑是理所应当的。但是其实技术发展到现在，很多技术（虽然不是全部）已经可以做到你中有我，互通有无了。所以在现时已经可以这样问这个问题：我们打算怎样**组合**这些技术？

### **Modern mobile strategy**

“怎样**组合**这些技术”是modern mobile strategies的基础，也是与old-fashioned strategies的主要区别。

我们使用native作为“粘合剂”来组合不同的技术，因为各种技术都能和native轻松沟通。大多数情况下，被组合的各种技术之间没有必要互相知道对方。因为更多时候，我们是利用不同技术的优势来实现app中的不同部分，是相对独立的。我们可以让native提供`Global Router`的服务，在各项技术需要相互沟通的时候提供协助。

可以用于组合的技术必须能够将自身融入到native的app里面，而不是渴望着独占一切。幸运的是，目前大多数选项都很谦卑。

最基本也是最开始被大家使用的是HTML。HTML最大的优势是富文本展示，做过iOS开发的dev都很清楚在iOS下做图文混排展示是很繁琐的，而HTML的强项就在这里。提到HTML很多人就会想到Cordova和基于它的一系列框架，其实我们完全不需要为了一些HTML页面而引入Cordova，即使需要在native和HTML之间有一些基本的交互，也可以通过众多js bridge库来实现。如果真的有模块需要大量的HTML页面和交互的话，Cordova也提供了CDVViewController用于将其嵌入到native app里。

这个领域目前最火热的React Native也保持了同样的接入方式。虽然使用官方命令行工具创建的React Native app把native的部分尽可能的隐藏了起来（就跟Cordova做的一样）。但是实际上，React Native的模块也是可以通过一个标准的ViewController加载进来的。更进一步，还可以通过RCTRootView来把React Native以subview的身份添加到Native的view hierarchy里。

所以modern mobile app的前端架构应该是：以native为基础，基于模块从不同深度选择符合“当前”产品策略和团队能力的方案混合进app中。
So the architecture of modern mobile app should be: based on native, choose and mix in different frameworks for different features based on the product strategy and team capability.

这样做的目的是为了在拥有足够开发能力的情况下，依然保留全身而退的可能*。
This is for keeping the ability

比如Facebook使用native技术开发app，当他们认为React Native已经足够成熟的时候，就可以很方便的选取某个相对独立的功能来作为React Native的切入点，并随着产品的演进进一步蚕食使用native开发的功能。
For example, Facebook builds some native apps. Whey they felt the React Native is good enough, they started some new feature with it, and build more feature with

随着时间的推进，未来很有可能会出现比RN更好的框架（从Facebook的开源史来看，他们很可能因此而放弃甚至删除RN库😂），然后团队决定拥抱新技术，那么只要新技术是可以在native上寄生的，就可以毫不费力的集成进来；如果RN无法满足某个功能所要求的性能或依赖，或者该功能非常关键以至于团队愿意为最高效能买单，那么团队大可以退一步切换回native方式来完成该功能。
There would be some frameworks better than React Native in the future (based on the open source history of Facebook, they might remove the whole React Native library from GitHub in such time😂). If the team decides to move to the new technologies, they can integrate them easily as long as the new framework can be plugged in to the native base. Or if there are some features that can't be implemented by React Native within acceptable time frames. Or if the feature is very important so that the team want to keep it in the best. Then the team can just switch back to the native safely.

诚然，全面推倒重写也可以达成同样的目的，but the point is to **be able to** do that with a **minimal** effort.
Sure, This can be archived by rewriting the whole app throughly, but the point is to **be able to** do that with a **minimal** effort.

熟悉我的人都知道我从一开始就不推荐Xamarin，原因之一**是Xamarin采用的是“独占”模式，当RN这样的新技术出现时，对新技术的拥抱将有可能非常困难（我没说不可能）。并且当系统API发生较大变化的时候，开发团队的应对速度也不可能达到native级别。当然，如果有人坚信Microsoft的技术方案可以以原生速度适配iOS和Android的话，我也没什么好说的。

Just don't be slave of frameworks.

##### _Notes:_

##### _* 当然团队中需要至少拥有几名native专家。不过即使选择其他策略，认为非native团队不需要native专家也绝不是好实践。我会在以后的文章里讨论这个问题。_

##### _** 也有很多别的原因，比如真实的代码重用比例很低，比如开发人员对系统API要达到native级别的精通等。如有需要我也会写专文讨论。_
