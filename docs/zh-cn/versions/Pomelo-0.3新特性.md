#Pomelo 0.3版本新特性

Pomelo 0.3版为移动端性能优化做了很多工作， 新协议的数据包压缩后的传输量仅为0.2版的20%， 并保留了0.2版基于socket.io的传输协议。socket.io对开发浏览器端器端的实时应用非常适合，而socket（websocket）、protobuf、二进制等协议则对移动端、桌面客户端的开发更具优势。pomelo对它们的同时支持使同时支持浏览器、移动、桌面客户端的高实时应用或游戏变得非常容易。

服务器的动态扩展是另一个重要的特性， 这不仅使系统适应了弹性的工作环境，也为游戏的一些动态功能提供了更多方便， 如MMORPG游戏的动态副本等。

0.3版还提供了其它很多特性，如新的广播接口， 新的客户端支持等。

##1 新协议支持

0.3版Pomelo开始支持二进制协议，并支持对请求route的字典压缩和请求内容进行protobuf压缩。0.3版同时兼容以前版本基于socket.io的通讯协议。通过在应用中配置不同的connector component来实现协议的切换或共存。

目前Pomelo服务器提供两类connector：sioconnector和hybridconnector，分别对于基于socket.io和二进制的通讯。

###1.1 sioconnector

支持基于socket.io的通讯协议，也是Pomelo框架默认采用的connector（主要是兼容老版本）。之前基于socket.io的服务器和客户端代码不用修改就可以使用。

###1.2 hybridconnector

支持socket和websocket，使用二进制通讯协议，并且支持route字典压缩和protobuf压缩的connector，需要在app.js中显式配置。以下是一个hybridconnector的配置例子：

```
app.configure('production|development', 'connector', function(){
  app.set('connectorConfig', {
    connector: pomelo.connectors.hybridconnector,
    heartbeat: 3,
    useDict: true,
    useProtobuf: true,
    checkClient: function(type, version) {
    	// check the client type and version then return true or false
    },
    handshake: function(msg, cb){
      cb(null, {/* message pass to client in handshake phase */});
    }
  });
});
```

首先，通过`app.set('connectorConfig', opts)`更改了connector组件的默认配置。`opts`是最终将被传递给connector，不同的connector可以有不同的opts内容。

+ connector - 指定所采用的connector实现，用来支持不同的底层通讯协议。Pomelo提供了`pomelo.connectors.sioconnector`和`pomelo.connectors.hybridconnector`。开发者也可以实现自己的connector，以支持不同的通讯协议。
+ heartbeat - 客户端与服务器的心跳间隔。不指定则表示不需要心跳。具体心跳流程在协议文档心跳小结介绍。
+ useDict - 是否进行route字典压缩，默认为false。具体字典压缩原理请参考协议文档字典压缩小结。
+ useProtobuf - 是否对协议内容进行protobuf压缩，默认为false。protobuf的流程请参考协议文档protobuf压缩小结。
+ checkClient - 可选的客户端鉴定函数。如果设置该字段，则要求客户端在握手阶段*必须*传递其版本号给服务器，并通过此函数来鉴定该版本客户端是否适用。如果该函数的返回值为false，则拒绝该客户端的后续操作。
+ handshake - 可选的握手函数，在客户端和服务器握手过程中被调用，用来在握手阶段向客户端传递自定义的信息。

###1.3 不同connector的共存

不同的connector可以在同一个项目中共存，支持不同的客户端连接，只要配置不同的前端服务器，监听不同的端口即可。例如：

```
// frontend server based on socket.io
app.configure('production|development', 'sio-connector', function(){
  app.set('connectorConfig', {
    connector: pomelo.connectors.sioconnector
  });
});

// frontend server based on socket and websocket
app.configure('production|development', 'hybrid-connector', function(){
  app.set('connectorConfig', {
    connector: pomelo.connectors.hybridconnector
  });
});

```

servers.json的配置

```
{
  "development": {
    "sio-connector": [
      {"id": "sio-server-1", "host": "127.0.0.1", "port": 3150, "clientPort": 3010, "frontend": true}
    ],
    "hybrid-connector": [
      {"id": "hybrid-server-1", "host": "127.0.0.1", "port": 3250, "clientPort": 3020, "frontend": true}
    ]
  }
}

```

关于Pomelo二进制协议的具体格式，请参考[Pomelo协议文档](https://github.com/NetEase/pomelo/wiki/Pomelo-%E9%80%9A%E8%AE%AF%E5%8D%8F%E8%AE%AE)，字典压缩和protobuf压缩相关内容请参考[Pomelo数据压缩协议](https://github.com/NetEase/pomelo/wiki/Pomelo-%E6%95%B0%E6%8D%AE%E5%8E%8B%E7%BC%A9%E5%8D%8F%E8%AE%AE)。

##2 动态服务器扩展
Pomelo 0.3版开始支持动态增加和移除服务器进程机制，并提供相应的命令行工具。每台新增的服务器都会连接到master服务器上进行注册。Master再将新服务器的信息广播给集群中的所有服务器进程。原有的服务器进程再对新增服务器事件进行响应。

当一个服务器进程接收到一个新增服务器的消息后，会将该服务器信息保存到本地的app上下文中，之后可以通过`app.getServers`等系列方法查看到新服务器的信息，从而影响之后的消息路由。

如果新加的服务器的类型之前尚未存在于app上下文中，Pomelo会尝试着为其创建对应的rpc代理对象。如果该类型服务器需要提供rpc服务，则需要在约定的目录（servers/server-type/remote/）下提供rpc服务代码。

动态移除服务器进程的流程也与上面类似，当服务器断开与master的连接后，master会将该服务器的信息广播给其他进程，其他进程再进行相应处理。

*注意*: Pomelo仅提供动态增加和移除服务器进程的机制，但由此而导致的路由规则的变化应当由具体的应用自己来维护。需要谨慎处理好诸如数据状态等问题。

###命令行支持

Pomelo 0.3版本的命令行工具根据新特性增加了一个add命令，同时对之前的stop命令做了一定的修改。

pomelo add命令主要作用是动态的添加服务器，暂时不支持前端服务器的添加(0.3后面的版本可能会支持)。命令需要的参数有四个，分别是服务器地址，服务器端口号，服务器id（用来标识服务器）,服务器类型。具体的命令格式如下：

```
pomelo add host=[host] port=[port] id=[id] serverType=[serverType]
```

pomelo stop命令主要作用是动态的停止服务器，可以动态的停止某一台服务器或者同时停止多台服务器。命令主要需要的参数就是服务器id, 可以同时有多个id,这样就是停止多台服务器，如果没有id，默认就是停止所有的服务器。具体的命令格式如下：

```
pomelo stop [id]
```

##3 其他新特性

###3.1 servers.json配置文件的修改

随着Pomelo支持协议的增多，配置文件原先定义的`wsPort`（代表websocket port）已不适用，现调整为`clientPort`。并新增一个字段frontend来标识该服务器是否是前端服务器，前端服务器 *必须* 设置该字段为true，后端服务器可以不设置或设置成false。

以[lordofpomelo](https://github.com/NetEase/lordofpomelo)为例，[servers.js](https://github.com/NetEase/lordofpomelo/blob/master/game-server/config/servers.json)文件修改如下。

connector服务器配置：

```
"connector": [
  {"id": "connector-server-1", "host": "127.0.0.1", "port": 3150, "clientPort": 3010, "frontend": true},
  {"id": "connector-server-2", "host": "127.0.0.1", "port": 3151, "clientPort":3011, "frontend": true}
]
```

gate服务器配置：

```
"gate": [
  {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true}
]
```

服务器代码中有依赖到配置信息`wsPort`的地方也需要做相应的调整。[lordofpomelo](https://github.com/NetEase/lordofpomelo)中的[gateHandler](https://github.com/NetEase/lordofpomelo/blob/master/game-server/app/servers/gate/handler/gateHandler.js#L29)中的`res.wsPort`需要改为`res.clientPort`。

###3.2 新增channel广播接口

`ChannelService`新增`broadcast`接口，适用于给全世界广播的场景。该接口会将消息推送到指定类型的所有前端服务器进程，再由后者将消息推送给连接的所有客户端。详细接口描述请参考[Pomelo API](http://pomelo.netease.com/api.html)。

使用例子：

```
var frontendType = 'connector';
var route = 'test.hello';
var msg = {msg: 'hello world'};
var opts = {binded: true};
app.get('channelService').broadcast(frontendType, route, msg, opts, function(err) {
  // check the broadcast result
});
```

###3.3 新增session获取接口

`LocalSessionService`新增根据session id获取`localSession`接口`get`和根据user id获取`localSession`接口`getByUid`。

使用例子：

```
var frontendId = 'connector-server-1';
var sid = 1;
var uid = '123456';
app.get('localSessionService').get(frontendId, sid, function(err, localSession) {
  // do something with the local session
});

app.get('localSessionService').getByUid(frontendId, uid, function(err, localSession) {
  // do something with the local session
});

```

###3.4  javascript客户端支持

#### 3.4.1 支持两类javascript客户端 --- socket.io与websocket
目前socket.io与websocket两套协议在javascript都有用武之地。socket.io的兼容好，可以适应各种浏览器，适合开发类似聊天室这样的高实时应用。websocket客户端则在数据压缩上做到了很多优化，大大减少了消息的传输量，适合开发基于HTML 5的游戏应用。

两个客户端的发布地址如下：

* [websocket客户端](https://github.com/pomelonode/pomelo-jsclient-websocket)
* [socket.io客户端](https://github.com/pomelonode/pomelo-jsclient-socket.io)

#### 3.4.2 客户端javscript build系统

以前的pomelo-jsclient版本管理异常混乱， 经常将好几个文件复制到不同的项目各自修改。因此我们引入了[component](https://github.com/component/component)来管理js的库。
客户端js代码目前采用以[component](https://github.com/component/component)的形式统一管理与维护。
因此在lordofpomelo的web-server下面的js库代码就只剩下，component.json，local文件夹，以及colorbox文件夹  
其中，component.json包含了指向local这个文件夹所指定的本地component，用于在component build的时候作为初始化操作  
而local文件夹里面有一个boot component，里面对lordofpomelo需要用到的客户端js库进行了配置，配置类似于npm的配置方式  

```  
{  
  "name": "boot",  
  "description": "Main app boot component",  
  "dependencies": {  
    "component/emitter":"master",  
    "NetEase/pomelo-protocol": "0.3.x",  
    "pomelonode/pomelo-protobuf": "*",  
    "pomelonode/pomelo-jsclient-websocket": "master",  
    "component/jquery": "*"  
  },  
  "scripts": ["index.js"]  
}    
```  

使用component之前，需要进行build，为了便于开发，在web-server目录的bin文件夹下面，有一个 component.sh 
web-server目录下执行命令，即可完成install和build客户端代码的工作  
```  
sh bin/component.sh  
```  

如果你只是修改了本地的component而不需要install线上的版本，那么你可以在web-server目录下执行命令  

```  
sh bin/build-component.sh  
```  

即可对本地的component进行build操作，而不去线上install  

具体详情还请到 [component](https://github.com/component/component) 上去了解  

###  3.5 其它客户端支持

* 提供新的C客户端，支持socket协议，基于libuv开发，提供了完整的数据与消息压缩， 支持cocos2d-x
* 其余客户端，如flash,android, ios, unity3d， 目前还只支持socket.io协议，后续会推出基于socket协议客户端