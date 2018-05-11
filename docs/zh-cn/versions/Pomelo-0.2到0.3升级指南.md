##servers.json配置文件的修改

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

##connector配置

0.2中基于socket.io connector的服务器代码可以无需修改就能使用。

如要使用支持socket和websocket的connector，在app.js配置指定使用hybridconnector即可。使用例子如下：

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

更详细的信息和多connector的共存请参考[Pomelo 0.3新特性](https://github.com/NetEase/pomelo/wiki/Pomelo-0.3%E6%96%B0%E7%89%B9%E6%80%A7)文档。

##route字典压缩配置
route字典压缩支持在传输过程中对route字段进行压缩，这一修改对用户是完全透明的，用户可以在不修改任何代码的情况下应用这一功能。只要在app.js中加入配置开关就可以启用字典压缩功能，具体配置如下：
```
  app.set('connectorConfig', {
    connector: pomelo.connectors.hybridconnector,
    heartbeat: 3,
    useDict: true,
    handshake: function(msg, cb){
      cb(null, {/* message pass to client in handshake phase */});
    }
  });
```
其中“useDict：true”项就会打开字典压缩功能，之后所有的系统生成route在传输时就会自动进行压缩，而这一过程对用户是完全透明的。
对于用户自定义的route，需要在项目中加入/game-server/config/dictionary.json文件，在其中定义需要压缩的route，系统在传输时会自动进行替换，以[lordofpomelo](https://github.com/NetEase/lordofpomelo)为例，dictionary.json的格式如下：
```
[
	"onDropItem",
	"onAttack",
	"onDied",
	"onMove",
	"onUpgrage",
	"onPickItem",
	"onRevive",
	"addEntities",
	"onRemoveEntities",
	"onPathCheckout"
]
```
关于route压缩的更多内容，见以[Pomelo 压缩协议](https://github.com/NetEase/pomelo/wiki/Pomelo-%E6%95%B0%E6%8D%AE%E5%8E%8B%E7%BC%A9%E5%8D%8F%E8%AE%AE)。
##protobuf编码配置
protobuf消息内容的编码格式，与之前的json相比，可以大大的减少数据的传输量。在lordofpomelo的测试中，编码后的消息大小只有json格式的20%左右。
pomelo中的porotobuf编码与之前是完全兼容的，可以在不修改代码的情况下直接开启这一功能，只需要在app.js中配置开启并加入消息的proto文件即可，app.js中的配置如下：
```
  app.set('connectorConfig', {
    connector: pomelo.connectors.hybridconnector,
    heartbeat: 3,
    useProtobuf: true,
    handshake: function(msg, cb){
      cb(null, {/* message pass to client in handshake phase */});
    }
  });
```
其中的“useProtobuf：true”选项就会开启protobuf编码功能。
要对某个消息进行编码，只需要在pomelo指定文件中加入对应消息的protobuf编码就可以了，默认的protobuf编码文件位置是 /game-sever/config/serverProtos.json 和 /game-server/config/clientProtos.json，分别表示服务端->客户端的消息和客户端->服务端的消息，以[lordofpomelo](https://github.com/NetEase/lordofpomelo)为例，其内容格式如下：
```
"onMove" : {
    "required uInt32 entityId" : 1,
    "message Path": {
      "required uInt32 x" : 1,
      "required uInt32 y" : 2
    },
    "repeated Path path" : 2,
    "required uInt32 speed" : 3
  },
  "onAttack" : {
    "required uInt32 attacker" : 1,
    "required uInt32 target" : 2
  }
```
关于protobuf压缩的更多内容，见以[Pomelo 压缩协议](https://github.com/NetEase/pomelo/wiki/Pomelo-%E6%95%B0%E6%8D%AE%E5%8E%8B%E7%BC%A9%E5%8D%8F%E8%AE%AE)。
##web客户端build配置

客户端js代码目前采用以[component](https://github.com/component/component)的形式统一管理与维护。如果升级客户端代码则需要进行简单的配置，比如在lordofpomelo的web-server下面的js库代码就只剩下，component.json，local文件夹，以及colorbox文件夹  
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

其中index.js 则是对component的具体使用开发逻辑，在lordofpomelo中，我们只是把各个依赖的库给挂到windows作用域下面，以兼容老的代码    
如果你是新开发的话，完全可以利用component的规范采用类似node.js的开发方式来开发客户端js代码  

最后使用component之前，需要进行build，比如在lordofpomelo中，为了便于开发，在web-server目录的bin文件夹下面，有一个 component.sh 
web-server目录下执行命令，即可完成install和build客户端代码的工作  
```  
sh bin/component.sh  
```  

如果你只是修改了本地的component而不需要install线上的版本，那么你可以在web-server目录下执行命令  

```  
sh bin/build-component.sh  
```  

即可对本地的component进行build操作，而不去线上install  

最后你只需要把build完成之后的 build.js 引入到你的前端页面中即可  

具体详情还请到 [component](https://github.com/component/component) 上去了解  