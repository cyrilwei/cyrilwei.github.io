---
layout: post
title:  "How to build an iMessage app"
date:   2016/09/25 16:53:00 +0800
categories: iOS
published: false
---

With the release of iOS 10, 最耐玩的新功能应该就是iMessage app了。除了最基本的支持自定义Sticker外，真正吸引人的是可以内嵌app，除了一些基本的限制外，这些内嵌的app可以实现非常复杂的功能。

并且iMessage app有一个先天优势，就是它是基于用户已有的联系人，并且建立在一个成熟的消息系统中。这就让turn based的多人游戏拥有了绝佳的土壤，这种方式远比在Game Center里重新添加各种好友要方便，也比基于facebook等的OAuth，或者自己建立服务器要方便得多。试想不需要任何注册登录等繁琐的准备，就可以跟好朋友玩儿游戏，多么舒畅的体验啊。

创建Sticker app不需要任何代码，只需要把图片打包就可以了。所以这里主要讲一下iMessage app后面的一些机制。

iMessage app, 从本质上来说，是Apple Messages app的extensions。所以它遵循app extensions的基本规则。于此同时Apple提供了一个叫“Messages Framework”的库用来协助开发。

Xcode 8里提供了iMessage Application template。如果是创建全新的iMessage app的话，这是个不错的起点。

[iMessage app template.png]

如果是在已有的app中增加iMessage app的支持，只要在project中添加一个iMessage Extension target就可以啦。

[iMessage Extension target.png]

iMessage app可以独立存在也可以作为一个app extension within a containing app。作为独立app存在的时候，home screen上不会显示该app，只有在Messages的app list里才会显示。而作为一个app内嵌的extension的时候，会在两个地方都显示。

下面我们就结合Messages framework提供的功能来讲一下如何开发iMessage app。

### MSConversation class

`MSConversation`用于表示整个conversation，主要的属性是`localParticipantIdentifier`和`remoteParticipantIdentifiers`，后面remote的是个数组，表示conversation支持多人聊天。

`MSConversation`还提供了很重要的一组`insert`方法，用于将message，sticker等放到Messages app的input field里。嗯，只是放在input field里，并不发送，最后发送与否以及使用什么effect发送由用户决定。

[put into input field.png]

同时`MSConversation`还有一个`selectedMessage: MSMessage?`属性。对于交互型app来说，这也是一个很重要的属性，通过它可以获取别人发送过来的message的详细内容。

### MSMessage class

MSConversation
MSMessage
MSSession

MSMessagesAppViewController
MSMessageTemplateLayout

URLComponents
 message url: Optional(?Base=base01&Scoops=scoops03)
