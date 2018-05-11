---
title: 总结
---

<!-- TOC -->

- [未涉及到的](#未涉及到的)
    - [服务器监控与管理](#服务器监控与管理)
    - [插件机制](#插件机制)
    - [IDE选择](#ide选择)
    - [服务器的配置](#服务器的配置)
    - [connector 的选择](#connector-的选择)
    - [多客户端支持](#多客户端支持)
- [总结](#总结)

<!-- /TOC -->

到这里为止，pomelo 的基本核心特性都已经进行了示例，虽然都没有很深入。还有一些前面教程没有涉及到的地方，我们把它放到这里进行一些简单的介绍。

## 未涉及到的

### 服务器监控与管理

pomelo 提供了一个命令行工具，通过这个命令行工具，可以进行初始化一个 pomelo 项目，启动一个 pomelo 项目，查看当前已经启动的服务器信息，以及关闭服务器群。从 pomelo 0.6.0 开始，使用这个工具在进行服务器的查看以及关闭时，将需要一个身份验证,需要提供用户名和密码，其命令行格式如下：

```bash    
$ pomelo [list|stop|kill] [-u <username>][-p <password>]
```

如果不提供用户名和口令的话，会使用默认的，默认的用户名和口令都是admin。目前命令行工具仅仅保留了初始化项目，启动项目，关闭项目，以及查看启动的服务器信息。更多的服务器管理操作可通过使用 [pomelo-cli](Pomelo-cli使用) 来完成，pomelo-cli 提供了强大的服务器管理功能，支持动态增加关闭服务器，支持更详细的服务器信息监控等。

### 插件机制

pomelo 提供了基于插件的扩展机制，一个插件中可以包含一些相关的 component 以及对 pomelo 事件的响应处理，关于插件的更多介绍，请参阅 [plugin的参考文档](https://github.com/NetEase/pomelo/wiki/plugin%E6%96%87%E6%A1%A)。

### IDE选择

在开发中，往往需要一款顺手的 IDE。对于 pomelo 来说，[这里](https://github.com/NetEase/pomelo/wiki/%E4%BD%BF%E7%94%A8-WebStorm-IDE-%E8%B0%83%E8%AF%95-Pomelo-%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F)详细介绍了一款javascript的IDE的使用。

### 服务器的配置

在配置服务器信息的时候，我们看到，对于前端服务器，要给 frontend 配置为 `true`，同时除了要配置 rpc 使用的端口 port 外，还要配置供客户端连接的端口 clientPort。对于后端的服务器，则仅仅需要配置 rpc 调用的端口即可,后面的开发指南部分会有详细的介绍。

### connector 的选择

本教程中与客户端的连接使用了 hybridconnector，实际上 pomelo 的 connector 的是可定制的，目前还支持基于 socket.io 的版本，基于 socket.io 版本的chat例子在[这里](https://github.com/NetEase/chatofpomelo)。

### 多客户端支持

本教程中，客户端的选择了基于 websocket 的 web 客户端，理论上 pomelo 可以支持任何客户端，下面有部分 chat 客户端 demo 的链接:

* unity 客户端的 chat:
    * [socket 版本](https://github.com/HustLion/pomelo-chat-unity-socket) 
    * [socket.io 版本](https://github.com/NetEase/pomelo-unitychat)

* Flash 客户端的 chat Demo:
  [PomeloFlashDemo](https://github.com/mani95lisa/PomeloFlashDemo)

* Android 客户端

[https://github.com/NetEase/pomelo-androidchat](https://github.com/NetEase/pomelo-androidchat)

* IOS 客户端

[https://github.com/NetEase/pomelo-ioschat](https://github.com/NetEase/pomelo-ioschat)

* cocos2d-x 客户端

[https://github.com/NetEase/pomelo-cocos2dchat](https://github.com/NetEase/pomelo-cocos2dchat)

## 总结

通过 chat 这个例子，我们从最初的 chat 例子开始，一步一步对其进行更改，展示了 pomelo 的一些特性的用法，这里都没有涉及特别深，只是让用户明白如何去增加一个 filter，如何去增加一个 rpc 调用，如何启用 route 压缩和使用 protobuf 编码，如何定制自己的 component 并让pomelo 加载，如何定制一个 admin module 等。这些功能都是 pomelo 的核心功能。

通过这些个例子的学习，相信会对 pomelo 有更详细的了解，更加了解 pomelo 提供的功能，以及具体的使用方式。同时这个例子也可以作为用户使用 pomelo 的一个简单参考，它基本上涉及到了pomelo 的所有方面。