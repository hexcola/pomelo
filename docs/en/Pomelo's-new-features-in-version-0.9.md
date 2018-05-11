## Pomelo-rpc-zeromq

In pomelo 0.9, we provide another rpc module based on zeromq. Besides the origin pomelo-rpc, developers can choose pomelo-rpc-zeromq. And the comparison of performance test between the origin pomelo-rpc and the pomelo-rpc-zeromq refers to: 

[pomelo-rpc-zeromq performance test report](https://github.com/NetEase/pomelo/wiki/pomelo-rpc-zeromq%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A) 

[pomelo-rpc performance test report](https://github.com/NetEase/pomelo/wiki/pomelo-rpc%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A).

Usage：

1. install zeromq.

2. configure in app.js. Specific configuration is as follow:

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

Use example refers to  [chatofpomelo](https://github.com/NetEase/chatofpomelo/tree/zmq) zmq branch.

## Upgrade pomelo command line

According to netizens' suggestion, in pomelo 0.9 developers can restart servers and start servers by server type using command line. These commands are as follows:


pomelo start -t [server type]: start some type fo servers，if you would like to start servers by server type you need to start the master server first, for example:

```

pomelo start -t master
pomelo start -t connector

```

pomelo start -i [server id]: start a certain server，like above you must start the master server first，for example：

```

pomelo start -i master
pomelo start -i chat-server-1

```

pomelo restart: restart all other servers except the master server.

pomelo restart -t [server type]: restart some type of servers.

pomelo restart -i [server id]: restart a certain server.

```

pomelo restart -t connector
pomelo restart -i chat-server-1

```

## Pomelo-rpc callback timeout

Based on the issue[rpc callback overstock](http://nodejs.netease.com/topic/52b2d8470a516e1851b256e4), in new version of pomelo-rpc we add timeout for rpc callback. Rpc client records callbacks in application and adds corresponding timers, if rpc server receives the message and the corresponding timer will be cleared, otherwise the callback will be cleared if the message is not received in required time. And developers can set the callback timeout, default is 10s. The configuration is as follow：

```

app.set('proxyConfig', {
	timeout: 1000 * 20
});

```


## Blacklist for connections

In new version of pomelo we support adding blacklist for connections both statically and dynamically, so the server side can shield attacks from clients.

### Principle

When the connector server receives a connection from the client side, it will emit a connect event. And this event contains the client ip, the connector server will catch this event and invoke the blacklist function passed by developers. If the client ip is in blacklist, the server will kick the client.

### Usage

#### statically

You just need to pass a function to the connectorConfig which will return the blacklist by  callback function. This function needs a callback as its argument, and the callback function is like `function(err, list) {...}`. The usage example is as follow:

```

// ./game-server/app/util/blackList.js

var self = this;

self.blackList = ['192.168.100.1', '192.168.100.2'];

module.exports.blackListFun = function(cb) {

  cb(null, self.blackList);

};

```

```

// ./game-server/app.js

var blackList = require('./app/util/blackList');

app.configure('production|development', function() {

  app.set('connectorConfig', {

  	blacklistFun: blackList.blackListFun
  });

}

```

#### dynamically

Developers can add blacklist dynamically by pomelo-cli, and it supports specific ip and regular expression. The usage example is as follow:

```

blacklist 192.168.100.1
blacklist (([01]?d?d|2[0-4]d|25[0-5]).){3}([01]?d?d|2[0-4]d|25[0-5])

```

## Channel serialization interface

In pomelo 0.9 we provide channel serialization interface, developers can implement these interfaces to keep channels persistent. And when the server is restarted, channels will be restored in memory. Developers need to implement four methods:

#### add(key, value, cb)
add key value pairs

#### remove(key, value, cb)
remove key value pairs

#### load(key, cb)
load all values

#### removeAll(key, cb)
remove all values


Usage:

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

## hot restart

In new version of pomelo, we update the pomelo-rpc module. When the rpc server is broken, the rpc client will cache rpc requests from applications. When the rpc server is restored，the rpc requests will be flushed to the server side.

If there is only one-way dependency between different servers，it means only A type servers make rpc requests to B type servers, but do not have B type servers make rpc requests to A type servers. And B type servers do not have any instances, in this case B type servers can restart without any impacts.

Usage refers to[chatofpomelo](https://github.com/NetEase/chatofpomelo) store branch. In chatofpomelo only connector servers make rpc requests to chat servers.For channels exsit in chat servers, we use channel serialization interface to make them persistence.

Operation steps are as follows:

1. run pomelo start -t master， pomelo start -t connector, pomelo start -t gate, pomelo start -t chat in different terminals;
2. open web server，run chatofpomelo;
3. restart chat server;


## DecodeIO protobuf

In pomelo 0.9, we support another protobuf implementation from decodeIO. If you do not know it, you can refer to [decodeIO-protobufjs](https://github.com/dcodeIO/ProtoBuf.js).

###Usage

####Client

Use the latest pomelo-jsclient-websocket, and add pomelo-decodeIO-protobuf component in the client side, the component.js is as follow:


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

and the index.js is as follow：

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

####Server

In the server side you just need to use pomelo-protobuf-plugin, and the specific configuration is as follow：


```

app.configure('production|development', function() {

	app.use(protobuf, {
		protobuf: {
		}
	});
});

```

####Warning

Pomelo provides protobuf in previous version, and these protobufs cannot be used at the same time. It means that when you use pomelo-protobuf-plugin, you cannot open useProtobuf.

In consideration of consistence with the origin protobuf, pomelo 0.9 supports decodeIO-protobuf with serverProtos.json and clientProtos.json. Although we do not support *.proto in decodeIO-protobufjs， you can use tools provided by decodeIO-protobufjs to convert *.proto to *.json.


Usage example refers to [lordofpomelo](https://github.com/NetEase/lordofpomelo/tree/decodeIO_protobuf) decodeIO-protobuf branch.

## Pomelo websocket reconnect automatically

In pomelo 0.9，spomelo-jsclient-websocket can reconnect automatically. The client would reconnect when the connection lost for 5s, and the next reconnect delay time will double.For example the first reconnect delay time is 5s, the second one is 10s and so on. The default max reonnect attempts is 10, and this can be configured in app.js.

```

//reconnect configuration

pomelo.init({
		host: 127.0.0.1,
		port: 3050,
		reconnect: true
	}, function() {

});


//set max reconnect attempts

pomelo.init({
		host: 127.0.0.1,
		port: 3050,
		reconnect: true，
  		maxReconnectAttempts： 20
	}, function() {

});


```