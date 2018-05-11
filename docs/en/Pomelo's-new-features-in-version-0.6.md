# New features in Pomelo 0.6


Pomelo 0.6 is by far the biggest upgrade since version 0.3.

Pomelo 0.6 introduces plugin mechanism, and we make globalChannel and zookeeper as plugins for pomelo. Meanwhile, we provide a new interactive command-line tool which is quite convenient to do some maintenance operations. In addition, in order to enhance pomelo's safety, in pomelo 0.6 we import data signature and deal with illegal connections for hybridconnector. There are many other features, like more detailed rpc log , max connections for frontend server, servers reconnect after disconnecting from master and [pomelo-daemon](https://github.com/fantasyni/pomelo-daemon) etc.

## Interactive command-line tool

In order to make servers' maintenance much easier, we provide an interactive command-line tool. The detail information can be referred to [pomelo-cli](https://github.com/NetEase/pomelo-cli)  

## Server connection authorization  
server connect to master with authorization  
pomelo-admin provides a simple auth function in [pomelo-admin auth](https://github.com/NetEase/pomelo-admin/blob/master/lib/util/utils.js#L117)  
developers can provide self-defined auth in pomelo by  
in master server
```javascript
app.set('adminAuthServerMaster', function(msg, cb){
  if(auth success) {
    cb('ok');
  } else {
    cb('bad');
  }
})
```

in monitor server
```javascript
app.set('adminAuthServerMonitor', function(msg, cb){
  if(auth success) {
    cb('ok');
  } else {
    cb('bad');
  }
})
```

**note**: by default you should provide adminServer.json file under the **config** dir  
adminServer.json
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

**type** is the serverType, **token** is a string you can genrate by yourself  
when using in pomelo, you should fill all your servers with type:token  

## Plugin

In order to make the developers customize the framework, we introduces the plugin mechanism to pomelo. The detail information can be referred to [plugin](https://github.com/NetEase/pomelo/wiki/plugin%E6%96%87%E6%A1%A3).

## Connections encryption

In pomelo 0.6, we provide data signature for hybridconnector. The following is how it works:

* Firstly, the client generates rsa key pair and it would keep the private key; 
* Then the server would get the public key during the handshake phrase
* Thirdly, the client would send the message and the signature which is signed by the private key in the client side to the server
* Finally, the server would verify the signature with the public key from the client. The detail flow is as following chart:

![rsa](http://pomelo.netease.com/resource/documentImage/rsa.png)

### How to use
You need to add an option 'encrypt' on both client and server as below:

client
```javascript```
pomelo.init({
 host:'127.0.0.1',
 port:3014,
 encrypt:true
}, function() {
// do something connected
});
```

server
```javascript```
app.set('connectorConfig', {
 connector: pomelo.connectors.hybridconnector,
 heartbeat: 3,
 useDict: true,
 useProtobuf: true,
 useCrypto: true
});
```

## Illegal connections

In pomelo sioconnector is based on socket.io, and socket.io itself has dealt with illegal connections. For hybridconnector is based on socket which does not do anything about illegal connections, we have dealt with this problem in pomelo 0.6, including empty connections and connections which do not meet with pomelo protocol standard.

## Rpc debug log

According to pomelo developers' suggestion, we add more logs in pomelo-rpc. Developers only need to add app.enable('rpcDebugLog') in app.js, and modify game-server/config/servers.json as follow code:

```json```
{
 "type": "file",
 "filename": "./logs/rpc-debug-${opts:serverId}.log",
 "fileSize": 1048576,
 "layout": {
  "type": "basic"
 },
 "backups":5,
 "category": "rpc-debug"
}
```

## Overload protection

We have toobusy module to protect servers from overload in pomelo, and in new edition we add a feature to limit the max connections of frontend server. When the frontend server's connections exceed the max connections, it would refuse new connections. The configure code is as follow:

```json```
{"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort":3050, "frontend":true, "max-connections": 100}
```

## Servers reconnect after disconnecting

In distributed environment, there exists situations that servers(not master) disconnect from intranet. When the server recovers from disconnecting, it would also has problems for the whole system. So we have solved this problem in pomelo 0.6.

## Daemon start mode
We provide [pomelo-daemon](https://github.com/NetEase/pomelo-daemon) for pomelo services to deploy better in distributed enviroment, besides, pomelo-daemon provides rpc debug log collector to sync log files to mongodb. The detail information can refer to [pomelo-daemon](https://github.com/NetEase/pomelo-daemon)

## Updated logger

we update new version of logger.
- logger supports prefix appended ahead of log, prefix can be filename, serverId, host etc  
- logger support output line in debug environment  
- in pomelo, logger will output to appender named by **pomelo**  
The detail information can refer to [pomelo-daemon](https://github.com/NetEase/pomelo-logger) 

## pomelo-protobuf provides support for rootMsg defined protos  
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

In this example, Path, Equipment is root message. It can be reused in the whole definition.
With rootMsg defined in protos, developers can write proto files easier. The detail information can refer to [pomelo-protobuf](https://github.com/pomelonode/pomelo-protobuf#rootmessage-support)