## pomelo rpc支持zeromq通信

在pomelo 0.9中提供了基于zmq的rpc调用，开发者可以根据需要选择原有的pomelo-rpc或者pomelo-rpc-zeromq。基于zeromq和原有的pomelo-rpc的性能对比测试结果可以参考：

具体使用方法：

1. 安装zeromq

2. 在app.js中进行配置，具体配置如下所示：

```

var zmq = require('pomelo-rpc-zeromq');

app.configure('production|development', function() {

	app.set('proxyConfig', {
		rpcClient: zmq.client
	});

	app.set('remoteConfig', {
		rpcServer: zmq.server
	});

});

```

具体使用示例可以参考 [chatofpomelo](https://github.com/NetEase/chatofpomelo/tree/zmq) zmq分支

[pomelo-rpc-zeromq性能测试报告](https://github.com/NetEase/pomelo/wiki/pomelo-rpc-zeromq%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A)与原有的[pomelo-rpc的性能测试报告](https://github.com/NetEase/pomelo/wiki/pomelo-rpc%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A)可以参考。


## pomelo命令行改进

根据网友的建议，在pomelo 0.9版本中增加了重启的命令和服务器单独启动的命令；具体命令如下：


pomelo start -t [server type] 启动某类型的服务器，如果需要分开启动不同类型服务器时候使用，首先必须启动master服务器，例如：

```

pomelo start -t master
pomelo start -t connector

```

pomelo start -i [server id] 启动具体某个服务器，同上首先需要启动master，master服务器无需提供具体id, 例如：

```

pomelo start -i master
pomelo start -i chat-server-1

```

pomelo restart 重启除了master以为的其它服务器，可能会出现重启失败的情况。

pomelo restart -t [server type] 重启某类型的服务器，不包括master服务器

pomelo restart -i [server id] 重启具体某个服务器，不包括master服务器

```

pomelo restart -t connector
pomelo restart -i chat-server-1

```

## pomelo rpc增加回调超时机制

针对之前网友提出的[rpc回调函数积压问题](http://nodejs.netease.com/topic/52b2d8470a516e1851b256e4), 在新版本的pomelo-rpc中通过增加了回调函数的超时机制解决，rpc客户端在记录应用层的回调函数的同时添加对应的定时器，如果rpc服务端收到对应的消息则将定时器清除，如果定时器超时则将对应的回调函数清除。定时器的超时时间开发者可以进行设置，默认是10s， 具体使用如下：

```

app.set('proxyConfig', {
	timeout: 1000 * 20
});

```


## pomelo支持连接黑名单机制

在新版本的pomelo中连接服务器支持静态或者动态添加ip黑名单功能，服务端可以在连接服务器中增加黑名单对攻击的ip进行屏蔽。

### 原理

connector每接受一个连接都会抛出一个连接事件, 这个事件中含有该连接的客户端IP. connector会捕获该连接事件, 并调用用户传入的获取IP黑名单的函数, 如果该客户端IP在黑名单中, 则立刻将对应的socket断开. 以此来实现连接服务器的黑名单过滤功能.

### 使用方法

#### 静态添加黑名单

使用时只需要向在connector的connectionConfig配置中传入一个获取IP黑名单的函数即可, 这个函数需要接受一个回调函数作为其参数, 该回调函数形如`function(err, list) {...}`. 在获取IP黑名单的函数内, 拿到IP黑名单时(该黑名单应为一维`JS Array`), 以类似于`cb(null, self.list)`的形式调用IP过滤回调函数，具体使用方法如下：

```
./game-server/app/util/blackList.js
... ...
var self = this;
self.blackList = ['192.168.100.1', '192.168.100.2'];
module.exports.blackListFun = function(cb) {
  cb(null, self.blackList);
};
... ...
```

```
./game-server/app.js
var blackList = require('./app/util/blackList');
... ...
app.configure('production|development', function() {
... ...
  app.set('connectorConfig', {
  	blacklistFun: blackList.blackListFun
  });
... ...
}
```

#### 动态添加黑名单

动态添加黑名单可以通过pomelo-cli完成，其中运行输入具体ip或者正则表达式，具体命令如下：

```

blacklist 192.168.100.1
blacklist (([01]?d?d|2[0-4]d|25[0-5]).){3}([01]?d?d|2[0-4]d|25[0-5])

```

## channel 序列化接口

在新版本的pomelo中提供channel的序列化接口，开发者可以通过实现该接口将系统中创建的channel进行保存；同时当服务器重新启动后，系统会将之前保存的channel恢复到系统中。开发者需要实现如下四个接口：

#### add(key, value, cb)
add key value pairs

#### remove(key, value, cb)
remove key value pairs

#### load(key, cb)
load all values

#### removeAll(key, cb)
remove all values


具体的使用方法如下所示：

```

var store = require('./store');

app.set('channelConfig', {
			store : store,
			prefix : 'pomelo'
		});

```

```
//store.js

var redis = require('redis');

var StoreManager = function() {
  this.redis = redis.createClient(6379, '127.0.0.1', {});
};

module.exports = new StoreManager();

StoreManager.prototype.add = function(key, value, cb) {
  this.redis.sadd(key, value, function(err) {
    cb(err);
  });
};

StoreManager.prototype.remove = function(key, value, cb) {
  this.redis.srem(key, value, function(err) {
  	cb(err);
  });
};

StoreManager.prototype.load = function(key, cb) {
	this.redis.smembers(key, function(err, list) {
		cb(err, list);
	});
};

StoreManager.prototype.removeAll = function(key, cb) {
  this.redis.del(key, function(err) {
    cb(err);
  });
};

```

## 热重启的部分支持

在新版本的pomelo中对rpc模块进行了改进，在rpc服务端断开连接后，上层应用的rpc请求会在rpc客户端缓存；当rpc服务端恢复后，再次发起rpc请求时，会把之前的rpc请求一起发到rpc服务端。

在系统中如果是rpc单向依赖，也就是说系统中只有A类服务器发送rpc请求到B类服务器，没有B类服务器发送rpc请求到A类服务器，同时B类服务器是没有任何状态信息和实例化信息，这样B类服务器就可以在pomelo 0.9版本中重启，且不会影响系统的正常运行。

具体可以参考[chatofpomelo](https://github.com/NetEase/chatofpomelo) store分支。在chatofpomelo中只有connector到chat服务器的单向rpc依赖，对于chat服务器有channel的实例存在，所以使用channel序列化接口将channel存储，所以需要使用到redis。

操作步骤如下：

1. 在不同的命令行界面分别执行 pomelo start -t master， pomelo start -t connector, pomelo start -t gate, pomelo start -t chat;
2. 打开web服务器，运行chatofpomelo;
3. 关闭chat服务器，并重新启动；


## pomelo支持decodeIO protobuf

在pomelo 0.9版本中提供了对decodeIO的protobuf的支持，对于decodeIO的protobuf的介绍可以参考[decodeIO-protobufjs](https://github.com/dcodeIO/ProtoBuf.js).

###使用方法

####客户端

使用最新的pomelo-jsclient-websocket, 同时在客户端添加命名为pomelo-decodeIO-protobuf的component，并将其挂载到window对象下。对应的component.js如下所示：


```

{
  "name": "boot",
  "description": "Main app boot component",
  "dependencies": {
    "component/emitter":"master",
    "NetEase/pomelo-protocol": "master",
    "pomelonode/pomelo-decodeIO-protobuf": "master",
    "pomelonode/pomelo-jsclient-websocket": "master",
    "component/jquery": "*"
  },
  "scripts": ["index.js"]
}


```

对应的index.js如下所示：

```

  var Emitter = require('emitter');
  window.EventEmitter = Emitter;

  var protocol = require('pomelo-protocol');
  window.Protocol = protocol;

  var protobuf = require('pomelo-decodeIO-protobuf');
  window.decodeIO_protobuf = protobuf; 
  
  var pomelo = require('pomelo-jsclient-websocket');
  window.pomelo = pomelo;

  var jquery = require('jquery');
  window.$ = jquery;


```

####服务端

在服务端需要使用pomelo-protobuf-plugin，并在app.js中使用对应的插件，具体配置如下：


```

app.configure('production|development', function() {
	app.use(protobuf, {
		protobuf: {
		}
	});
});

```

####注意事项

pomelo原有的protobuf和decodeIO-protobufjs不能同时使用，即不能同时使用pomelo-protobuf-plugin插件并在前端服务器开启useProtobuf。

考虑到与原有的protobuf保持一致，pomelo 0.9版本中支持的decodeIO-protobuf同样采用serverProtos.json和clientProtos.json，不支持decodeIO-protobufjs中的*.proto格式，对于*.proto格式可以采用decodeIO-protobufjs提供的命令行工具转换成json格式。


具体的使用示例可以参考[lordofpomelo](https://github.com/NetEase/lordofpomelo/tree/decodeIO_protobuf) decodeIO-protobuf分支。

## pomelo websocket支持自动重连

在pomelo 0.9版本中，pomelo-jsclient-websocket 支持自动重连。重连发生在连接断开后的5s后，在重连失败后下次重连的时间将是上次重连时间的2倍；所以重连时间依次为5s, 10s，20s,依次类推。默认最大的重连次数是10次，该参数可以在连接初始化过程中进行配置。

```

//设置客户端重连

pomelo.init({
		host: 127.0.0.1,
		port: 3050,
		reconnect: true
	}, function() {

});


//设置客户端重连最大次数

pomelo.init({
		host: 127.0.0.1,
		port: 3050,
		reconnect: true，
  		maxReconnectAttempts： 20
	}, function() {

});


```