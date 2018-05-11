---
title: pomelo.js 入门
---

> 系统环境 Ubuntu 16.04

## 简介



## 准备

- 确保你的机器可以上网,因为安装pomelo的过程需要从网上下载其依赖的包。

- 确保你的系统上已经要安装了 `Node`

- 确保你的系统中安装有python(2.5 < version < 3.0)以及C++的编译器。Node的源码主要由C++代码和JavaScript代码构成，但是却用 `gyp` 工具来做源码的项目管理，该工具采用Python语言写成的。

- 虽然 `pomelo` 是用 `Javascript` 写成，但是 `pomelo` 依赖的库中，有使用了 `C++` 语言写的扩展，因此安装 `pomelo` 的过程中会使用到 `C++` 编译器。 

## 安装 pomelo

使用 npm 全局安装

```bash
npm install pomelo -g
```

使用源代码方式安装

```bash
$ git clone https://github.com/NetEase/pomelo.git
$ cd pomelo
$ npm install -g
```

## HelloWorld

1. **新建项目**

```bash
pomelo init ./HelloWrold
```

> 更多 [pomelo 命令行工具](https://github.com/NetEase/pomelo/wiki/pomelo%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%B7%A5%E5%85%B7%E4%BD%BF%E7%94%A8)

初始化项目的时候需要选择底层的通信协议： `socket.io` 或者 `websocket`，推荐选择 `socket.io`

默认的用户名密码都是 `admin`

2. **安装依赖包**

```bash
cd HelloWorld
sh npm-install.sh
```

这时候，我们可以观察一下项目结构:

```
HelloWorld/
    |-- game-server/
    |   |-- app/
    |   |-- config/
    |   |-- logs/
    |   |-- node_modules/
    |   |-- app.js
    |   |-- package.json
    |
    |-- shared/
    |
    |-- web-server/
    |   |-- bin/
    |   |-- public/
    |   |       |-- css/
    |   |       |-- image/
    |   |       |-- js/
    |   |       |-- index.html
    |   |-- app.js
    |   |-- package.json
    |
    |-- npm-install.bat
    |-- npm-install.sh

```


该目录结构很清楚地展示了游戏项目的前后端分层结构，分别在各个目录下填写相关代码，即可快速开发游戏。下面对各个目录进行简要分析：

- **game-server**

    `game-server` 是用 `pomelo` 框架搭建的游戏服务器，以文件 `app.js` 作为入口，运行游戏的所有逻辑和功能。在接下来的开发中，所有游戏逻辑、功能、配置等都在该目录下进行。

    - **app子目录** - 放置所有的游戏服务器代码的地方，用户在这里实现不同类型的服务器，添加对应的 `Handler`，`Remote` 等等。

    - **config子目录** 包括了游戏服务器的所有配置信息。配置信息以JSON文件的格式进行定义，包含有日志、master、server等服务器的配置信息。该目录还可以进行扩展，对数据库配置信息、地图信息和数值表等信息进行定义。总而言之，这里是放着所有游戏服务器相关的配置信息的地方。

    - **logs子目录** 日志是项目中不可或缺的，可以对项目的运行情况进行很好的备份，也是系统运维的参考数据之一，logs存放了游戏服务器所有的日志信息。

- **shared**

    shared存放一些前后端、`game-server` 与 `web-server` 共用代码，由于都是javascript代码，那么对于一些工具或者算法代码，就可以前后端共用，极大地提高了代码重用性。

- **web-server**

    `web-server` 是用 `express 3.x` 框架搭建的 `web` 服务器，以文件 `app.js` 作为入口，当然开发者可以选择Nginx等其他web服务器。如果游戏的客户端不是web的话，如Android平台的话，这个目录就不是必须的了。当然，在这个例子中，我们的客户端是web，所以web服务器还是必须的。


3. **启动项目**

首先启动游戏服务器：

```bash
cd game-server
pomelo
```

然后启动 web 服务器

```bash
cd web-server
pomelo
```

启动完成后，推荐使用 Chrome 打开 `http://localhost:3001`，就可打开页面，点击 `Test Game Server` 如果弹出 `game server is ok`. 说明运行成功：

![](../_asset/image/helloworld_test_snapshot.png)

4. **查看服务器运行情况**

我们可以使用 `pomelo list` 来查看已经启动的服务器：

![](../_asset/image/pomelo_list.png)

服务器状态可以查看5种状态信息：

- `serverId`：服务器的 `serverId`，同 `config` 配置表中的 `id`。
- `serverType`：服务器的 `serverType`，同 `game-server/config`目录下的 master.json 和 servers.json 文件中的配置信息 。
- `pid`：服务器对应的进程 `pid` 。
- `heapUsed`：该服务器已经使用的堆大小（单位：兆）。
- `uptime`：该服务器启动时长（单位：分钟）。

5. **关闭项目**

```bash
pomelo stop
```

## 小结

到这里为止，我们已经成功安装了 `pomelo`，并成功运行了 `HelloWorld`。接下来，建议你看一下 `pomelo` 整体的一个较详细的概述。 如果你已经迫不及待地想写代码，可以去 `pomelo` 例子教程, 那里以一个 `chat` 应用为例，一步一步地向你展示如何来使用 `pomelo` 进行一个实际应用的开发，以及 `pomelo` 的一些 `API` 的使用方式等。