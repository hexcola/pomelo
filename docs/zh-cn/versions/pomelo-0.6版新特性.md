在Pomelo 0.6版本中，对pomelo部分结构进行了调整，将原有的globalChannel组件和master高可用组件从框架中移出，以一种插件的形式提供给用户，具体可以参考[pomelo-globalchannel-plugin](https://github.com/NetEase/pomelo-globalchannel-plugin), [pomelo-masterha-plugin](https://github.com/NetEase/pomelo-masterha-plugin)；同时在新版本中提供了一个交互式命令行工具，相比之前的命令行工具，使用更为方便、安全，对于项目的运维会更加方便；另外为了进一步提高pomelo的安全性，在0.6版本中引入了数据的签名验证，同时对hybridconnector的非法连接进行了相关处理；其它新的特性还包括详细的rpc debug日志和前端服务器的基本过载保护功能。
##交互式命令行
为了方便开发者更好的对使用pomelo开发的服务进行运维，在新版本中提供了交互式命令行工具，具体情况可以参考[pomelo-cli](https://github.com/NetEase/pomelo-cli)  
##服务器连接认证  
服务器与master之间的连接需要进行认证以提高服务的安全性，目前在pomelo-admin中提供了一个简单的服务器认证，可以看 [admin auth](https://github.com/NetEase/pomelo-admin/blob/master/lib/util/utils.js#L117)   
使用连接认证，需要在 config 目录下添加 adminServer.json 的文件  
```
[{
    "type": "connector",
    "token": "agarxhqb98rpajloaxn34ga8xrunpagkjwlaw3ruxnpaagl29w4rxn"
}, {
    "type": "chat",
    "token": "agarxhqb98rpajloaxn34ga8xrunpagkjwlaw3ruxnpaagl29w4rxn"
},{
    "type": "gate",
    "token": "agarxhqb98rpajloaxn34ga8xrunpagkjwlaw3ruxnpaagl29w4rxn"
}
]
```
**type** 是serverType, **token** 是一个字符串，你可以自己生成  
你可以通过自己定义的认证还是来完成认证的工作  
具体情况可以参考 [admin auth server](https://github.com/NetEase/pomelo-admin#server-master-auth)  

##plugin机制
为了方便开发者根据自身的需求对pomelo原有的功能进行有效的扩展，在新版本中提供了插件机制的功能，同时将之前版本中的globalChannel和master服务器高可用部分以plugin的形式提供，具体情况可以参考 [pomelo plugin](https://github.com/NetEase/pomelo/wiki/plugin%E6%96%87%E6%A1%A3)。如果要将pomelo升级到0.6，对于之前的pomelo-sync库需要重新修改在app.js中的配置，参考配置修改如下：
```
var sync = require('pomelo-sync-plugin');
app.use(sync, {sync: {
 // opts parameters passed to constructor
}});
```
##连接加密
在pomelo 0.6版本中，对hybridconnector增加了数据签名的功能。客户端首先产生rsa的密钥对，客户端保留rsa私钥，在握手阶段客户端将公钥发到服务端；在发送消息阶段，客户端使用私钥对消息进行签名，客户端将消息和签名一起发送到服务端，服务端进行签名验证，如果验证成功后面的流程继续，如果验证失败则该数据包则不进行处理。具体流程可以参考下图：

![rsa](http://pomelo.netease.com/resource/documentImage/rsa.png)

###使用方法
在客户端连接的过程中增加encrypt:true，在服务端app.js中配置useCrypt，具体代码参考：

客户端
```javascript```
pomelo.init({
 host:'127.0.0.1',
 port:3014,
 encrypt:true
}, function() {
// do something connected
});
```

服务端
```javascript```
app.set('connectorConfig', {
 connector: pomelo.connectors.hybridconnector,
 heartbeat: 3,
 useDict: true,
 useProtobuf: true,
 useCrypto: true
});
```

##非法连接处理
在pomelo中sioconnector是基于socket.io，socket.io本身是有对非socket.io的连接进行处理的。对于hybridconnector，底层是基于socket的，在pomelo 0.6版本中增加了对不符合规定协议的连接进行拒绝处理。主要包括两个部分：1.对空连接进行了超时处理；2.对不符合协议规范的连接进行拒绝处理。
##rpc debug日志
根据网友的建议，在pomelo 0.6版本中增加了更多的rpc日志。开发者只需要在app.js中使用app.enable('rpcDebugLog')即可，另外需要在game-server/config/servers.json中配置category为rpc-debug的appender，具体配置可以参考如下代码：

```json```
{
 "type": "file",
 "filename": "./logs/rpc-debug-${opts:serverId}.log",
 "maxLogSize": 1048576,
 "layout": {
  "type": "basic"
 },
 "backups":5,
 "category": "rpc-debug"
}
```

##过载保护
在pomelo之前的版本中有toobusy模块对服务器进行过载保护，在新版本中增加了一个对connector连接数的限制功能，开发者只需要在servers.json中对不同的connector进行最大连接数量的配置，当connector超过配置的最大数量，服务器会拒绝连接。配置可以参考如下代码：

```json```
{"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort":3050, "frontend":true, "max-connections": 100}
```

##服务器断网重连机制
在之前pomelo的版本中，在分布式环境下，存在服务器(非master)网络短时间断开，然后网络恢复后服务无法恢复的情况。在pomelo 0.6版本中，如果服务器(非master)短时间断开，网络恢复后服务器可以正常工作。

##Daemon启动模式
pomelo-daemon提供对pomelo服务进行分布式环境下的启动以及rpc debug日志的收集  
详细情况请看 [pomelo-daemon](https://github.com/NetEase/pomelo-daemon)

##升级的logger
在新版中对pomelo中的logger进行了升级  
- logger支持自定义prefix输出，把prefix打印在消息的头部，prefix可以是文件名，serverId, host 等等
- logger在debug模式下支持log行号的打印，方便开发者分析调试  
- 在pomelo中的日志统一输出到了category为pomelo的appenders   
详细情况请看 [pomelo-logger](https://github.com/NetEase/pomelo-logger)

##pomelo-protobuf支持rootMsg的proto文件定义  
通过定义rootMsg，可以很好的进行复用，简化protos文件的大小  
```
{
  "message Path": {
    "required double x" : 1,
    "required double y" : 2
  },
  "message Equipment" : {
    "required uInt32 entityId" : 1,
    "required uInt32 kindId" : 2
  },
  "onMove" : {
    "required uInt32 entityId" : 1,
    "repeated Path path" : 2,
    "required float speed" : 3
  },
  "area.playerHandler.enterScene" : {
    "message Player" : {
      "message Bag" : {
        "message Item" : {
          "required uInt32 id" : 1,
          "optional string type" : 2
        },
        "repeated Item items" : 1
      },
      "required uInt32 entityId" : 1,
      "required uInt32 kindId" : 2,
      "required Bag bag" : 3,
      "repeated Equipment equipments" : 4
    },
    "optional Player curPlayer" : 2
  }
}
```
以上定义的Equipment和Path都是可能复用的root message。
详细情况情况 [pomelo-protobuf](https://github.com/pomelonode/pomelo-protobuf#rootmessage-support)