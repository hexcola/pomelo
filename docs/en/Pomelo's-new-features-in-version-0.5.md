# New features in Pomelo 0.5

Pomelo 0.5 enhances the high availability of servers，including the HA of the master server and a configurable auto-restart for other servers. In addition, this version supplies the globalChannel component and the ability to bind processes to specified CPUs.

## Master HA
Although Pomelo is a distributed server framework, the master server is as a global controller a single player. In order to improve the availability of integrated service in Pomelo, we import zookeeper to implement the HA of the master server.
Since the HA of the master server is implemented by zookeeper, the availability of the zookeeper service itself needs to be as high as possible.
### Principle
The master server uses the hot standby technology, which uses one machine as a master and others as slaves. When the master goes down，one of the slaves will act as the new master, and connect with other servers to provide services. All these procedures are automatically applied.
### Instruction
If you want to use the HA of the master server, you just need to configure app.js as follows：
```javascript```
app.configure('production|development', function(){
	app.enable('masterHA');
	app.set('masterHAConfig',
		{
			server : '127.0.0.1:2181',
			path : '/pomelo/master'
		});
});
```
app.enable('masterHA') means opening the ability of master HA, at this time the server will load zookeeper client and try to connect the server(127.0.0.1:2181) which configured in app.js. Developers can use app.set('masterHAConfig') to configure zookeeper, in which server represents zookeeper server and path represents the node that zookeeper uses. This node must be visited normally.

There are two ways to start slave server：
* The first way is to modify /config/master.json，using different ip and port, then start it. At this time, the system knows it is a slave server so other services will not be started repeatedly.
* The second way is to start it with the command line tool. [pomelo add command](https://github.com/NetEase/pomelo/wiki/Command-Line-Document)

## globalChannel

globalChannel provides global channel service, which implements by storing information in redis. Developers can replace redis with other dbs; Pomelo channelService can only create channel in one specified server, and the channel can only store users' information in this server. The globalChannelService can create global channel, which can store users' information in any server.

### Instruction

globalChannel will not load as default，if you want to use, just configure it in app.js as follows:(redis-server opened)

```javascript```
app.configure('production|development', function(){
	app.set('globalChannelConfig',
		{
			host: '127.0.0.1',
		        port: 6379
		});
});
```

you can get the globalChannelService from application object like app.get('globalChannelService'); if you want to know specific interfaces, please refer to [Pomelo API document](http://pomelo.netease.com/api.html).

## Servers auto restart

Pomelo 0.5 add the ability that servers(not master) can automatic restart after disconnect from master server. We suggest to developers that only servers without status can be configured as auto-restart, only in this way the system will not be disrupted by the restart of servers. Servers will not automatic restart as default, if you want to open this ability just configure servers.json as follows:

```json```
{
    "development":{
        "connector":[
            {"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort": 3050, "frontend": true}
         ]
        "chat":[
           {"id":"chat-server-1", "host":"127.0.0.1", "port":6050, "auto-restart": true}
        ]
        "gate":[
	   {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true}
	]
    }
}
```

## Binding Process with CPU

In order to use cpu resource fully, Pomelo 0.5 add the ability that developers can bind specific cpu with different processes, this ability can only be used in linux sysstem. If you want to bind specific cpu with process, just configure in servers.json as follows:

```json```
{
    "development":{
        "connector":[
             {"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort": 3050, "frontend": true, "cpu": 2}
         ]
        "chat":[
             {"id":"chat-server-1", "host":"127.0.0.1", "port":6050, "cpu": 1}
        ] 
        "gate":[
	     {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true, "cpu": 3}
	]
     }
}

```

## Before server stop hook

In order to add custom handler before servers stop for developers, Pomelo 0.5 add this kind of custom event of servers' lifecycle. Developers can use app.beforeStopHook add custom handler as follows:

```javascript```
app.configure('production|development', 'connector', function() {
          var fun = function(app, cb){
            //do something
            cb();
          }
          app.beforeStopHook(fun);
});
```