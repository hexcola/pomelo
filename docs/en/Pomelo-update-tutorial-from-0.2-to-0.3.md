## Modification fo server.json

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


##Configuration of connectors

The code based on socket.io in the previous version can be ported to 0.3 version without any modification.

To use the new connector which based on socket and websocket, just only need to add a simple configuration in the `app.js` as following:

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

More details of connectors, please refer to [New features of Pomelo 0.3](https://github.com/NetEase/pomelo/wiki/Pomelo-0.3-new-features).

##Configure route compression
Route compression support compress the route to a 2 bytes integer at transport time. User can open this function without change any code.  The below code is how to open the route compression function at app.js:

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
The 'userDict : true' means to open the route compression. The default route compression will compress all the system generated routes, like 'area.playerHandler.move' etc.

To add the dictionary for user defined route, like 'onMove', 'onAttack', you need to add all the use defined routes at the /game-server/config/dictionary.json file, the format of the file is as follow: 

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

For more information of route compress, see [pomelo data compression protocol](https://github.com/NetEase/pomelo/wiki/Pomelo-data-compression-protocol).

##Protobuf encode config
In pomelo 0.3, we support message compression based on protobuf. The protbuf encode protocol will reduce the message size dramaticly, comprae to JSON format. The average compressed rate is 20% in the demo lordofpomelo.
 
To use the protobuf fuction, you need to open it in app.js, the config is as follow:
```
	app.set('connectorConfig',
		{
			connector : pomelo.connectors.hybridconnector,
			heartbeat : 3,
			useProtobuf : true,
			handshake : function(msg, cb){
				cb(null, {});
			}
		});
```
The 'useProtobuf:true' is the flag, after set that, pomelo will transport the protos data to client, and open protobuf decompress function.

After set the protbuf flag in app.js, to encode a message with protobuf, all you need to do is to add the protobuf protos in the proto file. There are two porotos files in pomelo:  /game-sever/config/serverProtos.json and /game-server/config/clientProtos.json, which means Server->Client message and Client->Server messages, the content format is as follow:

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
To get more information about protobuf compression, you can see [Pomelo compression protocal](https://github.com/NetEase/pomelo/wiki/Pomelo-data-compression-protocol), and [Pomleo-protobuf module](https://github.com/pomelonode/pomelo-protobuf).

##Building system for Javascript client

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