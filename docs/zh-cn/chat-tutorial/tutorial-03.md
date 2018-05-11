---
title: chat源码下载及安装
---

在这一部分，我们来实现一个简易的分布式聊天应用，为了简单化，我们将直接从 github 上获取对应的源码，当然你也可以首先 `pomelo init`,获得一个初始的项目目录，然后参照 github 上的源码，建立相应的目录，在默认的位置填充相应的源码。


## 源码结构

源码在github上面，通过如下命令，获得：

```bash
$ git clone https://github.com/NetEase/chatofpomelo-websocket.git
$ cd chatofpomelo-websocket
$ git checkout tutorial-starter
```

这个是很简单的应用，其代码结构如下图：

![源码结构图](../_asset/image/chat_source.png)

#### game-server

game-server 目录放的是所有游戏服务器的逻辑，以文件 app.js 作为入口，运行游戏的所有逻辑和功能。从图上可以看出其 servers 里面有三个目录，分别是 gate，connector，chat。在 pomelo 中，使用路径来区分服务器类型，因此三个目录代表了三种不同类型的服务器，每一个目录下面可以定义 handler, remote,定义了 handler 和 remote 就决定了这个服务器的行为。

- 对于 gate 服务器，其逻辑实现代码在其 gateHandler.js 中，它接受客户端查询 connector 的请求，返回给客户端一个可以连接的 connector 的 `(ip,port)`;

- connector 服务器，其逻辑代码在 entryHandler.js 中，它主要完成接受客户端的请求，维护与客户端的连接，路由客户端的请求到 chat 服务器;

- chat 服务器，其既有 handler 代码，也有 remote 代码， handler 中处理用户的 send 请求，而 remote 是当有用户加入或者退出的时候，由 connector 来发起远程调用时调用的。在 remote 里由于涉及到用户的加入和退出，所以会有对 channel 的操作。

game-server 的子目录 config 下面是游戏服务器所用到的配置文件存放的地方，配置信息使用 JSON 格式，包含有日志，master 服务器和其他服务器的配置信息。除了这个 pomelo 所需的配置信息外，一般情况下，也将游戏逻辑所需要的配置信息放到这个目录下，例如数据库的配置信息，地图信息等。

logs 子目录下存放游戏服务器产生的所有的日志信息。 

#### web-server

由于我们这个聊天应用的客户端是 web，所以需要一个 web 服务器。在这个目录下，主要是客户端的 js，css 和静态资源等等。在本例子中，里面有用户登录，聊天的逻辑的 js 文件等等。我们在这个例子教程中，更多地关注的是服务器端的逻辑以及功能，对于客户端，我们几乎不需要怎么修改其代码，直接使用默认就好。

### 安装及运行

首先，确保你已经成功安装了 pomelo。执行命令安装依赖:

```bash
$ sh npm-install.sh
```

启动游戏服务器:

```bash
$ cd game-server
$ pomelo start
```

启动web服务器:

```bash
$ cd web-server
$ node app.js
```

如果启动过程中没有问题的话，下面我们就可以使用我们的聊天服务了，打开浏览器，输入`http://127.0.0.1:3001/index.html`, 输入一个用户名和一个房间名，就可以加入到聊天中了。可以多开几个客户端实例，测试chat是否能正常地运行，可以在一个房间里广播，也可以单个给某一个人发消息，效果图如下： 

![chat](../_asset/image/chatofpomelo.png)

## chat分析

我们要搭建的pomelo聊天室具有如下的运行架构：

 ![multi chat](../_asset/image/multi-chat.png)

在这个架构里，前端服务器也就是 connector 专门负责承载连接， 后端的聊天服务器则是处理具体逻辑的地方。

这样扩展的运行架构具有如下优势：

- 负载分离：这种架构将承载连接的逻辑与后端的业务处理逻辑完全分离，这样做是非常必要的， 尤其是广播密集型应用（例如游戏和聊天）。密集的广播与网络通讯会占掉大量的资源，经过分离后业务逻辑的处理能力就不再受广播的影响。

- 切换简便：因为有了前、后端两层的架构，用户可以任意切换频道或房间都不需要重连前端的 websocket。

- 扩展性好：用户数的扩展可以通过增加 connector 进程的数量来支撑。频道的扩展可以通过哈希分区等算法负载均衡到多台聊天服务器上。理论上这个架构可以实现频道和用户的无限扩展。

#### 客户端

聊天室的逻辑包括以下几个部分：

- 用户进入聊天室：这部分逻辑负责把用户信息注册到 session，并让用户加入聊天室的 channel。
- 用户发起聊天： 这部分包括了用户从客户端发起请求，服务端接收请求等功能。
- 广播用户的聊天： 所有在同一个聊天室的客户端收到请求并显示聊天内容。
- 用户退出： 这部分需要做一些清理工作，包括 session 和 channel 的清理。

客户端首先要给 gate 服务器查询一个 connector 服务器，gate 给其回复一个 connector 的地址及端口号，这里没有列出完整的代码，具体的代码在路径 `web-server/public/js/client.js` 中,详细代码略去，见`client.js` :

```js
function queryEntry(uid, callback) {
  var route = 'gate.gateHandler.queryEntry';
  // ...
}

$("#login").click(function() {
  username = $("#loginUser").attr("value");
  rid = $('#channelList').val();

  // ...

 //query entry of connection
  queryEntry(username, function(host, port) {
    pomelo.init({
      host: host,
      port: port,
      log: true
    }, function() {
              // ...
    });
  });
});

```

客户端在查询到 connector 后，需要发请求给 connector 服务器， 第一次请求要给 connector 进程，因为首次进入时需要绑定对应的 uid 信息，这里略去详细代码:

```js
pomelo.request('connector.entryHandler.enter', {username: username, rid: rid}, function(){
  // ...
}); 
```

当用户发起聊天的时候，会请求服务 `chat.chatHandler.send`，大致代码如下:

```js
pomelo.request('chat.chatHandler.send', {content:msg, from: username, target: msg.target}, function(data) {
  // ...
});
```
当有用户加入、离开以及发起聊天时，同房间的人将会收到服务端推送来的相应消息,这些在客户端是以回调的方式进行添加的，大致代码如下：

```js

pomelo.on('onAdd', function(data) {
  // ...
});

pomelo.on('onLeave', function(data) {
  // ...
});

pomelo.on('onChat', function(data) {
  // ...
});

```

客户端的详细代码都在目录 `web-server/public/js/client.js` 文件中，这里，客户端的 js 是使用 [component](https://github.com/component/component) 进行管理的,详细请参阅 component 的参考文档。

### 服务端

我们知道，在 pomelo 中，只要定义了一个服务器的 handler 和 remote，那么就定义了这个服务器的行为，就决定了这个服务器的类型。在本例子中，有三种服务器，gate，connector，chat,它们完成的具体逻辑如下:

- gate 完成客户端对 connector 的查询，在其 handler 里有其实现的代码，由于在这里，本例中仅仅配置了一台 connector 服务器，因此直接返回其信息给客户端即可，然后客户端就可以连接到 connector 了。

```js
handler.queryEntry = function(msg, session, next) {
	var uid = msg.uid;
	// ...
};
```

- connector 接受用户的连接，完成用户的注册及绑定，维护客户端 session 信息，处理客户端的断开连接，其逻辑代码在 `connector/handler/entryHandler.js` 中。大致如下：

```js
handler.enter = function(msg, session, next) {
	var self = this;
	var rid = msg.rid;
	var uid = msg.username + '*' + rid
	var sessionService = self.app.get('sessionService');
  // .....
};
```

- chat 服务器是执行聊天逻辑的地方，它维护 channel 信息，一个房间就是一个 channel，一个 channel 里有多个用户，当有用户发起聊天的时候，就会将其内容广播到整个 channel。chat 服务器还会接受 connector 的远程调用，完成 channel 维护中的用户的加入以及离开，因此 chat 服务器不仅定义了 handler，还定义了remote 。当有客户端连接到 connector 上后，connector 会向 chat 发起远程过程调用，chat 会将登录的用户，加到对应的 channel 中，其大致代码为：

```js
// chatHandler.js
handler.send = function(msg, session, next) {
	var rid = session.get('rid');
	var username = session.uid.split('*')[0];
	// .....
};

// chatRemote.js
ChatRemote.prototype.add = function(uid, sid, name, flag, cb) {
	var channel = this.channelService.getChannel(name, flag);
};

ChatRemote.prototype.kick = function(uid, sid, name) {
	var channel = this.channelService.getChannel(name, false);
  // ...
};
```
> **注意** 在实现具体的 Handler 的时候，最后需要调用 next，其中 next 的签名为 `next(err, resp)`.如果没有出现错误，那么 err 为空即可；如果不是 request 请求，而是 notify 的话，则一样需要调用 next，此时 resp 参数是不需要的，一般情况下，如果没有错误的话，就直接使用 `next(null)` 即可。

服务器配置信息在 config 目录下，现在我们只关注 servers.json, master.json。master.json 配置是master 服务器的配置信息，包括地址端口号，servers.json 配置具体的应用服务器信息。在配置文件中，分为development 和 production 两种环境，表示开发环境和产品环境，我们在 `pomelo start` 后面可以通过-e可以指定使用哪个环境，更多帮助参见 `pomelo start --help`。

## 小结

在这部分，我们下载了一个简单的聊天应用，并安装运行起来，并对其源码进行了分析。在本例子中，为了简单起见，我们对每一种类型仅仅配置一台服务器，其中对于前端服务器来说需要指定 frontend 为 true。[下一步](扩充服务器 "多台服务器")，我们将对每一种服务器类型配置多台服务器，以此来展示 pomelo 强大的可伸缩性。
