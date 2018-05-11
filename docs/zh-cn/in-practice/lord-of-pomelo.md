---
title: Lord Of Pomelo
---

<!-- TOC -->

- [介绍](#介绍)
    - [主要内容](#主要内容)
    - [整体架构](#整体架构)
    - [分析](#分析)
- [安装指南](#安装指南)
    - [运行环境](#运行环境)
    - [安装LordOfPomelo](#安装lordofpomelo)
        - [下载源码](#下载源码)
        - [安装依赖包](#安装依赖包)
        - [创建MySql数据库](#创建mysql数据库)
            - [创建数据库](#创建数据库)
            - [修改数据库配置](#修改数据库配置)
    - [运行游戏](#运行游戏)
    - [访问游戏](#访问游戏)
    - [相关问题解决办法](#相关问题解决办法)
    - [配置文件汇总说明](#配置文件汇总说明)
    - [登录问题说明](#登录问题说明)
- [服务器介绍](#服务器介绍)
    - [各类服务器介绍](#各类服务器介绍)
        - [connecter服务器](#connecter服务器)
        - [gate服务器](#gate服务器)
        - [验证服务器](#验证服务器)
        - [场景服务器](#场景服务器)
        - [寻路服务器](#寻路服务器)
        - [聊天服务器](#聊天服务器)
        - [副本/组队服务器](#副本组队服务器)
    - [场景管理与介绍](#场景管理与介绍)
        - [LordOfPomelo 中的场景管理](#lordofpomelo-中的场景管理)
            - [实体管理](#实体管理)
            - [消息服务](#消息服务)
            - [AOI服务](#aoi服务)
- [代码组织](#代码组织)
    - [game-server 服务端代码分析](#game-server-服务端代码分析)
        - [逻辑代码](#逻辑代码)
        - [服务器代码](#服务器代码)
    - [web-server 代码架构](#web-server-代码架构)
- [启动流程](#启动流程)
        - [组件的配置和加载](#组件的配置和加载)
        - [服务器的启动](#服务器的启动)

<!-- /TOC -->

# 介绍

[LordOfPomelo](https://github.com/NetEase/lordofpomelo)是一个基于Pomelo框架开发的分布式MMORPG游戏[Demo](http://pomelo.netease.com/lordofpomelo/).

LordOfPomelo是一个基于Pomelo游戏框架开发的MMORPG(大型多人在线角色扮演游戏)的游戏Demo, 具有角色、怪物、装备、战斗、聊天、技能、升级系统、任务系统、组队、副本等较为完整的游戏功能. 

LordOfPomelo服务端采用了Pomelo框架, 客户端采用了基于HTML5的colorbox框架, 在大约3个月的时间内实现快速开发. 服务端约8000行代码, 客户端约6000行代码. 支持单场景800人以上的并发访问, 响应时间控制在100ms左右. 

## 主要内容
涵盖了主流MMORPG的核心内容: 多个不同游戏场景, 两种职业, 多种类型的任务, 丰富多彩的道具和武器, 组队, 副本, 以及与各种怪物和Boss的战斗. 玩家可以在多个虚拟场景中穿梭, 完成各种任务, 提升等级, 并和其他玩家互动.

LordOfPomelo采用了基于分区的场景管理, 一张地图表示一个游戏场景, 与一个独立的场景服务器对应, 由该服务器提供对应的服务. 这种设计在避免了复杂的跨服事务的同时提供了对服务器扩展的支持, 开发者可以通过加入新的游戏场景来提高整个服务端的负载能力. 为了考察Pomelo服务器的响应能力, 我们在场景中采用了实时游戏模式: 玩家的攻击, 技能释放, 道具的检取、使用等行为都是实时进行的. 

整个游戏采用了Pomelo框架的标准开发模式, 实现了集群化服务器管理, 并且可以使用LordOfPomelo中对线性扩展的支持, 加入新的服务器来提高总体的负载能力. 在经过多轮性能测试与优化后, 达到了单场景800人的负载能力, 同时可以保证良好的响应时间(100ms左右). 

## 整体架构

![](../../assets/image/lord-of-pomelo-architecture.png)

如上图所示, LordOfPomelo包括两种类型的服务器: game-server和web-server. web-server是基于HTTP的web服务器, 玩家通过web-server实现注册和登录逻辑. 在玩家完成验证之后就会通过websocket连接到game-server集群, 进入实际的游戏场景之中. game-server是LordOfPomelo的核心服务器集群, 包括一组前端的websocket服务器, 以及后端的游戏逻辑服务器集群, game-server的架构如下图: 

![](../../assets/image/lord-of-pomelo-gameserver.png)

上图中的Client可以是任何支持websocket的客户端, LordOfPomelo中自带的客户端是通过HTML5实现的, 不但可以运行在PC的浏览器上, 而且可以运行在其它支持HTML5的终端中（如iPhone, iPad和配置较高的android设备）. 不同平台之间的玩家可以进行对等、实时的互动. 

如图所示, game-server中的服务器也分为两类: frontend server和backend server, frontend server是一组websocket服务器集群, 用来处理与websoket客户端之间消息通讯, 负责消息的转发, 过滤, 以及消息广播等功能. backend server则主要用来处理游戏逻辑, 包括各种不同类型的游戏逻辑服务器. 其中, 场景服务器是最重要的游戏服务器, 主要负责游戏场景管理, 游戏数据的更新和保存, 客户端请求的处理, 以及怪物和NPC行为的驱动等. 这些功能是通过与其他服务器的协同工作来实现的. 同时, 场景服务器也采取了可扩展的形式, 每个场景对应一个独立的场景服务器. 可以通过增加游戏场景来分散单个服务器的压力, 提高整体负载. 

## 分析

* [LordOfPomelo中的数据压缩](https://github.com/NetEase/pomelo/wiki/Pomelo-%E6%95%B0%E6%8D%AE%E5%8E%8B%E7%BC%A9%E5%8D%8F%E8%AE%AE)


-------------------------------

# 安装指南

## 运行环境
* [nodejs](http://nodejs.org/)
* Windows、Linux 或 MacOS 操作系统
* MySql 数据库

## 安装LordOfPomelo

### 下载源码

`git clone https://github.com/NetEase/lordofpomelo.git`

### 安装依赖包 

进入目录：
`cd lordofpomelo`

安装依赖包：
`sh npm-install.sh`(Windows: `npm-install.bat`)

### 创建MySql数据库

#### 创建数据库
sql文件路径：./game-server/config/schema/Pomelo.sql

* 安装MySql数据库(略)
* 登录MySql:
`mysql –u用户名 –p密码`
(登录成功提示符：mysql>)
* 创建数据库:
`mysql> create database Pomelo;`
* 选择数据库:
`mysql> use Pomelo;`
* 导入sql文件:
`mysql> source ./game-server/config/schema/Pomelo.sql`

#### 修改数据库配置
数据库配置文件为./shared/config/mysql.json
```json
{
  "development": {
   "host" : "127.0.0.1",
    "port" : "3306",
    "database" : "Pomelo",
    "user" : "xy",
    "password" : "dev"
  },
  "production": {
   "host" : "127.0.0.1",
    "port" : "3306",
    "database" : "Pomelo",
    "user" : "xy",
    "password" : "dev"
  }
}
```

将"development"环境下的的数据库配置修改为实际的配置. 

## 运行游戏

需要分别启动game-server和web-server. 
game-server的启动方式：

* `pomelo start` (pomelo的安装方法参考[pomelo快速使用指南](https://github.com/NetEase/pomelo/wiki/pomelo快速使用指南)) 注: 如果上次启动的进程没有完全退出, 可以使用`pomelo kill --force`来结束所有node进程. 

web-server的启动方式：

* `cd web-server && node app`

## 访问游戏
本地运行, 直接在浏览器中访问 http://localhost:3001 或者 http://127.0.0.1:3001

浏览器需支持websocket, 推荐使用chrome. 

## 相关问题解决办法
1. 端口冲突

修改服务器配置文件./game-server/config/servers.json,内容如下：
```json
{
  "development": {
    "connector": [
      {"id": "connector-server-1", "host": "127.0.0.1", "port": 3150, "clientPort": 3010, "frontend": true},
      {"id": "connector-server-2", "host": "127.0.0.1", "port": 3151, "clientPort":3011, "frontend": true}
    ],
    "area": [
      {"id": "area-server-1", "host": "127.0.0.1", "port": 3250, "area": 1}, 
      {"id": "area-server-2", "host": "127.0.0.1", "port": 3251, "area": 2}, 
      {"id": "area-server-3", "host": "127.0.0.1", "port": 3252, "area": 3}, 
      {"id": "instance-server-1", "host": "127.0.0.1", "port": 3260, "instance": true},
      {"id": "instance-server-2", "host": "127.0.0.1", "port": 3261, "instance": true},
      {"id": "instance-server-3", "host": "127.0.0.1", "port": 3262, "instance": true}
    ],
    "chat": [
      {"id":"chat-server-1","host":"127.0.0.1","port":3450}
    ],
    "path": [
      {"id": "path-server-1", "host": "127.0.0.1", "port": 3550}
    ],
    "auth": [
      {"id": "auth-server-1", "host": "127.0.0.1", "port": 3650}
    ],
    "gate": [
      {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true}
    ],
    "manager": [
      {"id":"manager-server-1","host":"127.0.0.1","port":3750}
    ]
  },
  "production": {
    // ...
  }
}
```

该配置文件分别定义了development和production环境下的各个服务器的配置信息, 包括服务器类型, 地址, 端口号等, production环境下的参数和development环境的结构类似. frontend参数为true时表示前端服务器. 端口冲突时可以修改相应的端口号. 

或者修改文件 ./web-server/public/js/config/config.js, 内容如下：

```js
__resources__["/config.js"] = {meta: {mimetype: "application/javascript"}, data: function(exports, require, module, __filename, __dirname) {
  module.exports = { 
    IMAGE_URL:'http://pomelo.netease.com/art/',
    GATE_HOST: window.location.hostname,
    GATE_PORT:3014
  };
}};
```

该文件是web服务器的常用配置信息: IMAGE_URL是图片等静态资源的地址; GATE_HOST是gate服务器接受websocket的入口, 所有的websocket请求须先经过gate服务器获取connector服务器的id, 然后再和connector建立连接; GATE_PORT是前端gate的端口号. manager服务器主要负责是副本和组队功能.

## 配置文件汇总说明

* ./game-server/config/master.json

master服务器的配置信息, 包括development和production环境下的服务器地址、端口号等. 

master服务器负责启动、关闭各服务器, 并监控所有服务器的状态信息. 
* ./game-server/config/servers.json

area、connector等服务器的配置信息, 包括development和production环境下的服务器地址、端口号等. 由于connector是前端服务器, 用于接收并转发玩家的请求, 所以会有clientPort. 

* ./shared/config/mysql.json

数据库配置信息, 在安装LordOfPomelo之后需要根据数据库安装的实际情况修改development和production环境的参数. 

* ./web-server/public/js/config/config.js

客户端图片等静态资源及HTTP访问地址的配置. 

## 登录问题说明

LordOfPomelo提供了注册新用户以及使用github, google, facebook, twitter和新浪微博授权的方式进行登录. 在使用github等授权方式进行登录时, 需要自己通过OAuth授权认证, 然后修改配置文件中的对应信息(./web-server/config/oauth.json). 

--------

# 服务器介绍

## 各类服务器介绍

LordOfPomelo采用分布式的设计, 服务端是由一个服务器集群组成的, 包括：多台场景服务器, 一台或多台寻路服务器, 聊天服务器, 副本/组队服务器, 全局服务器, 长连接服务器等. 

![backend server](../_asset/image/load-of-pomelo-backend-server.png)

### connecter服务器

与web的短连接模式不同, 在网络游戏中客户端与服务端建立的都是长连接. 而长连接本身是需要一定的资源来维持的, 在LordOfPomelo中, 我们使用websocket协议在客户端和服务器之间建立连接. 而connector服务器就是用来维护这些连接, 并中转客户端和服务端之间的消息. 在LordOfPomelo中, 客户端和服务端的连接状态是通过一个抽象的session来维护的, session是一个客户端在服务端的标识, 用来维护用户的登录状态, 用户的基本信息, 以及用户的websocket连接信息. 

### gate服务器

gate服务器的主要作用是为用户提供一个统一的websocket入口, 并负责用户验证和connector服务器的分配. 与connector服务器不同, LordOfPomelo中只有一台gate服务器. gate服务器会向所有客户端暴露一个固定的websocket接口, 当用户登录时, 会首先连接gate服务器, 完成验证, 并获得由gate服务器分配的对应connetor服务器的信息. 之后, 客户端会断开与gate服务器的连接, 通过获取的信息连接对应的connector服务器, 获取对应的服务. 

### 验证服务器

验证服务器负责用户注册和验证. 作为用户验证的统一入口, 提供远程调用接口供其他服务器调用, 来进行用户的验证. 验证服务器的主要作用是屏蔽认证验证的细节, 为其他服务器提供统一的验证接口. 

### 场景服务器

在网络游戏中, 出于性能和负载的考量, 大的游戏世界总是会被分成多个区域, 这些不同区域就是场景. 在LordOfPomelo中, 一张游戏地图就是一个游戏场景, 与一台独立的场景服务器对应. 场景是构成LordOfPomelo游戏世界的基本单位, 不能进行分割和简单的并行扩展. 场景服务器负责维护场景中所有实体, 并驱动实体AI运行游戏逻辑. 

场景服务器负责处理游戏中的几乎所有逻辑, 同时为其他服务器提供操纵场景数据的接口. 在LordOfPomelo中, 虽然场景本身不能分割, 但可以通过加入新的游戏场景的方法来分散用户, 从而提高游戏服务器总的负载量. 而一些与场景相关的服务也可以通过独立运行的方式进行水平扩展. 

### 寻路服务器

寻路服务是游戏服务器的基本服务之一, 玩家跑动, 怪物移动都需要寻路服务提供支持. 其功能是根据地图中的起点和终点, 得到一条这两点之间的最优路径. 由于寻路是典型的无状态、计算密集型服务, 在LordOfPomelo中, 我们将寻路逻辑与场景逻辑分离, 放在单独的寻路服务器中, 从而减轻了场景服务器的压力. 而寻路服务器也可以根据需要进行简单的并行扩展. LordOfPomelo中的寻路算法使用A*实现, 提供了通用的计算接口, 并封装为一个模块, 具体信息见[pomelo-pathfinding](https://github.com/NetEase/pomelo-pathfinding). 

### 聊天服务器

聊天服务是网游的基本服务之一. 在LordOfPomelo中, 聊天服务是与场景服务分离的, 通过一个独立的服务器来实现. 聊天服务器会维护一份所有在线用户的数据, 通过这些数据与connector服务器通讯, 来实现玩家之间的即时通讯. 

### 副本/组队服务器

副本/组队服务器是后端服务器集群中负责全局管理副本全生命周期和组队相关操作的功能服务器.具体可参考[lordofpomelo 0.3新特性](https://github.com/NetEase/pomelo/wiki/lordofpomelo-0.3%E6%96%B0%E7%89%B9%E6%80%A7) 

## 场景管理与介绍

LordOfPomelo中的场景服务负责管理游戏中所有实体和整个游戏世界. 而出于安全性和一致性等方面的考虑, 所有的游戏逻辑验证都是在后端进行的, 因此场景服务还包括了游戏中所有的逻辑判断和处理. LordOfPomelo的场景服务是游戏服务的核心, 也是逻辑最为复杂的部分, 下面就对LordOfPomelo中的场景服务进行分析：

### LordOfPomelo 中的场景管理

LordOfPomelo中每个场景对应一个独立的场景服务器, 所有的业务逻辑都在场景服务器内部进行. 对于跨场景的操作, 则是通过一个全局服务器来进行处理. 下图就是一台单独的场景服务器的功能：

![area](../_asset/image/load-of-pomelo-area.png)
 
#### 实体管理

游戏场景中的所有实体都会在进入场景时加载到内存中, 之后所有的修改都会直接在内存中进行, 并通过数据同步模块来定时同步到数据库中. 这一设计将场景中所有的数据操作都变成了直接的内存操作, 提高了整体性能, 规避了频繁的缓慢IO操作. 

LordOfPomelo中实体的定义都在domain目录下的entity文件夹中, 采用了面向对象的设计思想, 所有实体都继承自entity类. 实体的加入和删除都通过area中的接口(addEntity/removeEntity)来处理, 从而减轻了对象的管理负担. 

#### 消息服务

LordOfPomelo中的消息服务可以分为两种：一对一的RPC请求<-->响应, 以及一对多的广播消息. 
由于LordOfPomelo中的逻辑验证是在服务端进行的, 对于客户端的大部分请求, 服务端都会有一个直接的回复来进行处理, 这种回复是使用了类似于web中的request/response模式来实现的. 

对于广播服务, LordOfPomelo中使用了基于AOI的区域通知服务. 即在收到广播消息后, 根据消息类型, 通过AOI服务获得需要通知的玩家列表, 然后对该列表中的玩家进行广播, 从而大幅减少了消息的数量. 

#### AOI服务

玩家的视野一般远小于场景的大小, 因此对于场景中的绝大部分消息, 进行简单的全场景广播是没有必要而且无法承受的. 当有消息需要发送时, 如何确定该消息需要通知的玩家列表就是AOI模块的功能了. 

LordOfPomelo中实现的是基于灯塔的AOI服务. 其基本的实现方法是将整个地图划分为若干个等大的tower, 每个tower负责维护一个对象列表, 指向所有在这个tower范围内的对象. 然后在这个数据结构的基础上来触发各种AOI事件. 


------

# 代码组织

LordOfPomelo的代码主要包括两部分: 后端服务器代码game-server和前端客户端代码web-server. game-server是游戏服务端, 包括所有的游戏逻辑代码和游戏服务器代码. web-server是游戏客户端, 包括用户注册和登录界面代码, 以及一个用HTML5编写的游戏客户端. 除了这两部分之外, 还有一个公用的shared目录, 用来存放前后端共用的代码和配置. 

作为一个分布式的游戏服务器, LordOfPomelo可以同时运行在多台服务器上, 却统一使用同一套代码, 不同的服务器会根据配置加载各自的目录代码. 下图是LordOfPomelo的代码结构: 

![lordofpomelo_arch](../_asset/image/lord-of-pomelo-arch.png)

## game-server 服务端代码分析

![game-server](../_asset/image/lord-of-pomelo-game-server.png)

game-server根目录下的app.js是服务器代码的入口, 其他目录的功能如下: 
* /app    : 服务端js代码, 包括服务器代码和游戏逻辑代码. 
* /config : LordOfPomelo中的配置文件. 
* /logs   : 服务器端运行时产生的日志文件. 
* /scripts: 统计模块对应的本地脚本. 
* /test   : LordOfPomelo中的测试用例

### 逻辑代码

逻辑代码主要用来完成具体的业务逻辑, 如用来驱动怪物的AI代码, 用来计算地图中路径的寻路代码等, 逻辑代码在/app/domain目录下: 

![logic](../_asset/image/lord-of-pomelo-logic.png)

* /action : 负责处理客户端的请求. 由于场景是由tick驱动的, 而tick的间隔一般较短(默认100ms), 当请求需要在多个tick中执行的时候就会被封装为一个action来执行. 
* /aoi    : aoi相关逻辑, 包括aoi消息的封装, 以及对aoi消息的处理. 
* /area   : 场景相关逻辑, 提供场景中的主要接口. 包括: 场景中实体的加入、更新和删除, 广播消息的推送, 场景中服务的访问(AOI, AI等), 场景信息的获取等. 同时还包括一个Timer, 用来驱动场景中的逻辑. 
* /entity : 场景中的所有实体, 包括玩家, 怪物, npc, 宝物, 装备, 队伍等. 
* /event  : 用来集中处理场景逻辑中产生的各种事件, 包括玩家消息, 怪物消息等. 
* /map    : 用来完成地图的加载和解析, 以及地图中区域的抽象. 
* /task   : 任务相关的代码, 控制任务的执行和取消, 以及任务奖励的获得. 

### 服务器代码

![servers](../_asset/image/lord-of-pomelo-server-code.png)

服务器代码在/servers目录下, 通过规约的形式组织, 对外提供rpc接口, 处理客户端和服务端的请求并返回结果. LordOfPomelo中使用的服务器包括: 
* /area     : 场景服务器, 用来储存场景信息, 处理客户端的请求, 如用户添加, 删除, 攻击等操作. 
* /chat     : 聊天服务器, 处理聊天信息
* /connector: 连接服务器, 负责维护用户session, 接受用户数据, 并将服务端的广播数据推送给玩家
* /login    : 登录服务器, 用来验证用户登录信息
* /path     : 寻路服务器, 用来完成路径计算功能. 
* /manager  : 副本/组队服务器, 用来管理全局的副本和组队功能. 

## web-server 代码架构

![web server](../_asset/image/lord-of-pomelo-web-server.png)

LordOfPomelo的页面端代码主要分为两个部分: 基于HTML5开发的ui代码和使用colorbox开发的游戏逻辑代码. ui代码包括注册/登录页面, 游戏场景中的各种选项和菜单. 这些代码基于HTML5开发, 使用css3进行渲染. 游戏场景的绘制和游戏逻辑的驱动则是基于colorbox开发, 并使用到了HTML5中的很多特性. 

除此之外, web-server中还包括用户注册代码和oauth验证的逻辑, 这些代码在lib目录下. 

下图是页面端的内容: 

![client & html](../_asset/image/lord-of-pomelo-client.png)

* /animation_json : 动画相关的json描述.
* /css            : 代码中所用到的css文件. 
* /image          : 客户端中用到的图片资源.
* /js             : 所有客户端的js文件.
* /index.html     : 是LordOfPomelo的入口文件.

下图是游戏js代码组织: 

![client & html](../_asset/image/lord-of-pomelo-client-code.png)

* /config   : 客户端的配置信息.
* /handler  : 客户端的handler, 用来处理服务端response请求.
* /lib      : colorbox和pomelo的客户端通讯库代码.
* /model    : 客户端的游戏逻辑代码.
* /ui       : ui代码.
* /utils    : 客户端用到的工具类.
* /app.js   : 客户端的初始化入口, 负责初始化客户端的逻辑代码. 

-----------------

LordOfPomelo的启动流程采用了Pomelo中的启动模式. 在阅读下面内容前, 请先阅读[pomelo启动流程](https://github.com/NetEase/pomelo/wiki/pomelo%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B).


# 启动流程

app.js是LordOfPomelo的入口, 主要负责所有服务器的配置, 以及组件的加载和启动. LordOfPomelo的启动主要分为两步：启动master服务器, 再由master服务器分别启动其他服务器. 

### 组件的配置和加载
LordOfPomelo中使用了多个外部组件, 这些组件在服务器启动时加载, 提供各种服务：如数据统计, 路由功能的替换, 游戏场景的初始化等. 

LordOfPomelo中使用了基于脚本的统计, 这些组件通过运行自定义的脚本, 收集服务器运行数据并生成报告, 更加具体的功能, 请见：

``` javascript
	var sceneInfo = require('./app/modules/sceneInfo');
	var onlineUser = require('./app/modules/onlineUser');
	if(typeof app.registerAdmin === 'function'){
    app.registerAdmin('sceneInfo', new sceneInfo());
    app.registerAdmin('onlineUser',new onlineUser(app));
	}
```

LordOfPomelo启动时还会加载areas的配置文件, 用来建立场景和服务器之间的映射:

``` javascript
  //Set areasIdMap, a map from area id to serverId.
	if (app.serverType !== 'master') {
	  var areas = app.get('servers').area;
	  var areaIdMap = {};
	  for(var id in areas){
	  	areaIdMap[areas[id].area] = areas[id].id;
	  }
	  app.set('areaIdMap', areaIdMap);
	}
```

为了能在多个场景服务器中正确的路由, LordOfPomelo中加载了自定义的路由组件, 通过使用场景与服务器之间的映射信息, 可以确保玩家的请求被分发到对应的场景服务器上:

``` javascript
  // route configures
	app.route('area', routeUtil.area);
	app.route('connector', routeUtil.connector);
```

除了服务器的通用配置以外, app.js中还负责不同服务的初始化工作: 如全局服务器的初始化, 场景的初始化, 以及寻路服务器的初始化, 这些初始化会根据服务器的类型进行不同的初始化过程: 

``` javascript
app.configure('production|development', 'area', function(){
  app.filter(pomelo.filters.serial());
  app.before(playerFilter());
  //Load scene server and instance server
  var server = app.curServer;
  if(server.instance){
    instancePool.init(require('./config/instance.json'));
    app.areaManager = instancePool;
  }else{
    scene.init(dataApi.area.findById(server.area));
    app.areaManager = scene;
  }
  //Init areaService
  areaService.init();
});
```

数据同步插件和MySql的初始化:

``` javascript
// Configure database
app.configure('production|development', 'area|auth|connector|master', function() {
  var dbclient = require('./app/dao/mysql/mysql').init(app);
  app.set('dbclient', dbclient);
  app.use(sync, {sync: {path:__dirname + '/app/dao/mapping', dbclient: dbclient}});
}); 
```

### 服务器的启动

LordOfPomelo的启动也采用了Pomelo框架中的启动方式, 即将master作为一个默认组件, 在app.js调用app.start()方法后加载, 启动master服务. 

master组件会负责启动其他所有服务. 这个启动过程分为两个阶段：第一阶段, master服务启动其他所有服务器, 在服务器启动完毕后, 其中的monitor组件会连到master对应的监听端口上, 表明该服务器启动完毕. 第二阶段, 在所有服务器都启动完毕之后, mater会调用所有服务器上的afterStart接口, 来进行启动后的处理工作. 

