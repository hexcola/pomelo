#New features in Pomelo 0.3

Pomelo 0.3 improves the communication protocol between client and server for mobile enviroment. The new protocol uses compressive binary format and the size of the compressive package is only 20% of which in Pomelo 0.2.

Pomelo 0.3 also provides new connector to communicate with clients based on socket or websocket for socket(websocket), protobuf and binary protocol are more suitable with mobile and PC clients. Pomelo 0.3 keeps the socket.io connector at the same time since it is good for the web development. That means Pomelo 0.3 now supports the web, mobile and desktop clients at the same time and make it more simple and efficiently.

And dynamic scaling up servers is another important feature. It make Pomelo more suitable to the elastic computing system and online game, such as copy system.

There are other new features in Pomelo 0.3, such as new broadcast interface and new client SDK and so on.

##1 New protocol

We import a new binary protocol based on TCP or websocket in Pomelo 0.3 to support the route string compress and message body compress. And Pomelo 0.3 is also compatible with the older versions based on socket.io. Both the protocols can coexist in the same project only needing to make a little change on configure.

Currently, Pomelo framework provides two kinds of connector: sioconnector which is based on socket.io and hybridconnector which runs on TCP and websocket.

###1.1 sioconnector

`sioconnector` is the default connector of Pomelo which is compatible with the older versions. So the older Pomelo project can be ported to Pomelo 0.3 without modifying their codes. 

###1.2 hybridconnector

`hybridconnector` is the new connctor in Pomelo which is smaller in size and more efficient. To enable the `hybridconnector`, it only need add a little configure in the `app.js` as below.

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

First, the `app.set('connectorConfig', opts)` means customize the configuration of connector.

+ connector - specify the connector. The buildin connectors of Pomelo are `pomelo.connectors.sioconnector` and `pomelo.connectors.hybridconnector`. And the developers could also implement their own connectors to support more protocols.

+ heartbeat - heartbeat interval between client and server in second. Absence means no heartbeat. More details of heartbeat refers to [Pomelo protocol](#).

+ useDict - whether compress the route string by dictionary. Default value is `false`. More defails of dicionary compression refers to  [Pomelo compress](#).

+ useProtobuf - whether compress the message body with protobuf. Default value is `false`. More details of protobuf compresssion in Pomelo refers to [Pomelo compress](#).

+ checkClient - optional client verification function. If specified, the client *must* pass the version and type of the client to the server. And the server should verfiy and return a boolean value to figure out that the client with this version and type whether is suitable to connect to current server.

+ handshake - optional handshake function which would be fired in the handshake phace to let the server to pass some customized data to the client during handshake.

###1.3 Coexistence of connectors

More than one kinds of connector could be enabled in the same project to support different protocols in different ports as following:

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

servers.json

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

##2 Dynamic server extension

Pomelo 0.3 supports add and remove server processes dynamically and provides the associated command line tools. 

Each server would register itself to the master server when it startup. Then the master server broadcasts the new server joined event to the other servers in the cluster. And then all the other servers response to the event.

When a server process receives a new server joined event, it would add or update the new server information into the local app context. And then the other parts of the application could query the server information by `app.getServers` methods to make some changes, such as change the route searching.

If the type of the new server has not existed in the app context, Pomelo would try to create a RPC proxy. And if the application wants to provide some RPC servies for this type of server, please make sure deploy the relative code under the convention path (servers/server-type/remote/).


Remove a server process is similiar, the master detects a server disconnect event and broadcast to all the other servers. And the servers take response to the event.

*Notice*: Pomelo just provides the mechanism to add and remove server process dynamically. And the application should take the responsibility to take care status data during the processes changing.

###Command line

Pomelo 0.3 provides a new `add` command and upgrades the `stop` command to support add and remove a server process on the fly.

`pomelo add` is used to add a server process dynamically. The usage of `pomelo add` is as below. The `host` is the hostname or address of server, `port` is the port that server listens to, `id` is the id of server and the `serverType` is the type of server.

```
pomelo add host=[host] port=[port] id=[id] serverType=[serverType]

```
`pomelo stop` is used to stop one or a list of server processes. If no id specified, it would stop all the server processes by default.

```
pomelo stop [id]
```

##3 Other new features

###3.1 Modification fo server.json

`wsPort` in `servers.json` is replaced with `clientPort` since the protocol supported in Pomelo is not only websocket yet and the `wsPort` which means 'websocket port' is not suitable anymore. 

And a new field `frontend` is imported to indicate a server is whether a frontend server or backend server which is checked by the `wsPort` whether is specified in the previous versions. So for the frontend servers *MUST* set the `frontend` to true while the backend servers could set it to false or leave it empty by default.

For example, in [lordofpomelo](https://github.com/NetEase/lordofpomelo), the modification of [servers.js](https://github.com/NetEase/lordofpomelo/blob/master/game-server/config/servers.json) is as below:

connector server configuration：

```
"connector": [
  {"id": "connector-server-1", "host": "127.0.0.1", "port": 3150, "clientPort": 3010, "frontend": true},
  {"id": "connector-server-2", "host": "127.0.0.1", "port": 3151, "clientPort":3011, "frontend": true}
]
```

gate server configuration：

```
"gate": [
  {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true}
]
```

And remember to modify the dependecies of the `wsPort` in the codes. Such as replace the `res.wsPort` with `res.clientPort` in [gateHandler](https://github.com/NetEase/lordofpomelo/blob/master/game-server/app/servers/gate/handler/gateHandler.js#L29).

###3.2 New broadcast method for channel

`ChannelService` add a `broadcast` method for the global broadcast. This method would push the message to all the frontend servers with the specified type and the laters would push the message to all the connected client. More detail please refer to [Pomelo API](http://pomelo.netease.com/api.html).

Usage:

```
var frontendType = 'connector';
var route = 'test.hello';
var msg = {msg: 'hello world'};
var opts = {binded: true};
app.get('channelService').broadcast(frontendType, route, msg, opts, function(err) {
  // check the broadcast result
});
```

###3.3 LocalSession query method

`LocalSessionService` add new method to query the `loacalSession` instance by session id or by user id.

Usage:

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

###3.4 Javascript clients

#### 3.4.1 Types of Javascript client

There are two kinds of Javascript client in Pomelo: socket.io and websocket.

The socket.io version is good at compatible with all kinds of browsers and is suitable for the normal realtime web application, such chat room.

And the websocket version make a great efforts on the data compression and optimization. It is very helpful for the HTML5 games. 

The respositories of the two kinds of client:

* [websocket client](https://github.com/pomelonode/pomelo-jsclient-websocket)
* [socket.io client](https://github.com/pomelonode/pomelo-jsclient-socket.io)

#### 3.4.2 Building system for Javascript client

In Pomelo 0.3, we import the [component](https://github.com/component/component) system to manage the client Javascript libraries. And the client Javascript codes would be downloaded from `github` repository directly when building the client system.

The client building system is composed by the `component.json` description file and the `local/` directory under the `web-server/publilc/js/lib/`.

`component.json` in `public/js/lib/` describes the global component information of the client which points to the `local/` folder and contains a local component named `boot`.

Contents of `public/js/lib/component.json`

```
{
  "name": "pomelo-client",
  "description": "pomelo-client",
  "local": [ "boot" ],
  "paths": [ "local"]
}
```

And the `loacl/` folder contains the local components description. These is only one local component in the client system which named `boot` and located in `public/js/lib/boot/`. And the `boot` local component mainly includes two files: `component.json` and `index.js`.

The `boot/component.json` is a component description file like the one in `npm` and it mainly describes the base information and dependencies of the local component.

The conents of `boot/component.json`:

`component.json` is used to describe 

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

There are several tools located in `web-server/bin/` to help to build the client system.

`bin/component.sh` - install and build the client codes. It would fetch the latest codes from github and build the client.

Usage:

Input the command below in the `web-server` directory.

```  
sh bin/component.sh  
```  

`bin/build-component.sh` - build the client system. It would only build the client from the local files and would not fetch them from github.

For development, developers can modify the local files and build the new client by typing the following command in the `web-server` directory.


```  
sh bin/build-component.sh  
```  

More detail about component system, please refer to [component](https://github.com/component/component).

###3.5 Other clients supporting

* New C language client, based on `libuv` and `jansson`, supporting route string and message compression.
* Other clients, such as flash, android, IOS and unity3d, are available for socket.io version. And the socket versions are coming soon.