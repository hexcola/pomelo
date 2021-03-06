---
title: pomelo.js chat 教程
---


<!-- TOC -->

- [前言](#前言)
    - [热身](#热身)
    - [为什么是聊天?](#为什么是聊天)
    - [教程内容](#教程内容)

<!-- /TOC -->

# 前言

## 热身

- 本教程适用于对 `pomelo` 零基础的用户，如果你已经有过一定的 `pomelo` 开发基础，请跳过这个教程，你可以阅读开发指南，那里会对一些话题作较为详细的探讨。

- 由于 `pomelo` 是基于 `node.js` 并使用 `javascript` 开发的，因此希望你在阅读本教程前对 `node.js` 和 `javascript` 有一些了解。

- 本教程的示例源码放在 `github` 上，对于教程的不同部分，使用了不同的分支，因此希望你在使用本教程之前对`github` 有个简单的了解，能帮助你更好地使用本教程。

- 本教程将以一个实时聊天应用为例子，通过对这个应用进行不同的修改来展示 `pomelo` 框架的一些功能特性,让用户能大致了解 `pomelo`，熟悉并能够使用 `pomelo` 进行应用程序的开发。

- 本教程假定你使用的开发环境是类 `Unix` 系统，如果你使用的 `Windows` 系统，希望你能够知道相关的对应方式，比如一些.sh脚本，在 `Windows` 下会使用一个同名的 `bat` 文件，本教程中对于 Windows系统，不做特殊说明。

## 为什么是聊天?

Pomelo 是游戏服务器框架，本质上也是高实时、可扩展、多进程的应用框架。除了在提供的库部分有一部分游戏专用的库，其余部分框架完全可用于开发高实时的应用。而且与现在有的 `node.js` 高实时应用框架如 `derby`、`socketstream`、`meteor` 等比起来有更好的可伸缩性。

由于游戏在场景管理、客户端动画等方面有一定的复杂性，并不适合作为 pomelo 的入门应用。对于大多数开发者而言，node.js 的入门应用都是一个基于 socket.io 开发的普通聊天室， 由于它是基于单进程的 node.js 开发的， 在可扩展性上打了一定折扣。

因此，我们也选择做一个聊天应用来作为教程的例子，而基于 pomelo 框架开发的聊天应用天生就是多进程的，可以非常容易地扩展服务器类型和数量。

## 教程内容

在之前的章节中，我们已经看到了 HelloWorld 项目，对如何使用 pomelo 开发应用程序应该有了一个初步的印象。在这个教程里面，我们将以一个使用 pomelo 开发的实时聊天应用为示例，来依次展示如何在 pomelo 增加一个filter、进行 route 压缩、rpc 调用和使用 protobuf 等。这些例子仅仅是为了展示 pomelo 框架的特性，所以并不具有很强的实用性, 只是希望用户通过对这些例子的学习，可以熟悉 pomelo 各个特性的使用，给用户一个直观的认识,使得用户也能按照例子的方式，使用 pomelo 提供的功能。其大致内容包括以下部分：

- 我们首先会对 pomelo 中常见的一些术语进行一个简要的解释;
- 从 github 获得一个现成的分布式聊天应用，我们对其源码进行分析，然后让其运行起来;
- 对我们的聊天应用的服务器进行扩展，使用多个服务器来完成我们的应用; 
- 给原来的应用增加一个 filter，来实现我们想要的功能，同时学会 filter 在 pomelo 中的使用;
- pomelo 提供了 route 压缩，尝试在我们的应用里使用 route 压缩，同时学会如何在 pomelo 中使用 route 压缩功能;
- pomelo 提供了基于 protobuf 的消息压缩，尝试将其运用到我们的项目中,同时学会如何在 pomelo 中使用基于protobuf 的消息压缩;
- 增加一个获取时间的 rpc 服务，当然功能上纯粹属于“画蛇添足”，不过这里是出于演示如何使用 rpc 的目的;
- 尝试给 pomelo 增加一个 HelloWorld component，目的同上，只是为了演示如何给 pomelo 扩展组件，从而学会给 pomelo 扩展自定义组件;
- 增加一个 admin-module，以演示如何使用 admin-module 来扩展 pomelo 的服务器管理监控框架;
- 最后是关于 pomelo 的一些杂项以及总结。

当完成上面的教程后，相信你已经能够熟练使用pomelo提供的各项特性做一些开发了，这也是本教程要达到的目的。更深入的探讨可以参阅开发指南，或者直接阅读源码都是可以的。下面进入[下一步](术语解释 "术语解释")，我们将对pomelo中的一些术语进行简单的解释。
