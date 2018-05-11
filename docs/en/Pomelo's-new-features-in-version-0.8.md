# New feature in Pomelo 0.8

## Pomelo-cocos2d-jsb

In pomelo 0.8 we support cocos2d-x javascript binding, and developers can use javascript both in frontend and backend development. The detail infomation refers to [pomelo-cocos2d-jsb](https://github.com/NetEase/pomelo-cocos2d-jsb).

## Pomelo command line tool refactoring

#### Elimination for dependency on specified directory when executing pomelo command.

When executing `pomelo start` in a directory which is not the source code directory `game-server`, you need to indicate the location of `game-server` by the option `-d, --directory`. For `log4js` configuration file, as all the logs should be put into `game-server` directory, so it is necessary to specify it and ${opts: base} is used to do so. That means if you executed `pomelo start` in a non-`game-server` directory, you may need to use the new log4js configuration file and a template for it can be found in `template` directory. An example is as follows: 

```

{
  "appenders": [
    {
      "type": "console"
    },
    {
      "type": "file",
      "filename": "${opts:base}/logs/con-log-${opts:serverId}.log",
      "pattern": "connector",
      "maxLogSize": 1048576,
      "layout": {
        "type": "basic"
      },
      "backups": 5,
      "category": "con-log"
    },
    {
      "type": "file",
      "filename": "${opts:base}/logs/crash.log",
      "maxLogSize": 1048576,
      "layout": {
        "type": "basic"
      },
      "backups": 5,
      "category":"crash-log"
    },
    // other appenders...
  ],

  "levels": {
    "rpc-log" : "ERROR",
    "forward-log": "ERROR"
  },
  "replaceConsole": true,
  "lineDebug": false
}

```

For other pomelo commands, such as `pomelo list`, `pomelo add`, `pomelo stop`, it is needed to specify the address of `master` server, and it would be 127.0.0.1:3005 by default, that means all these commands can be executed remotely. Also, all these commands need authorization with a pair of `username` and `password`, the default `username` and `password` both are `admin` if you don't specify them.

For `pomelo masterha`, similar to `pomelo start`, it is also needed to specify the soure code location.

#### `pomelo start` improvement

Eliminating the dependency on `forever` if starting the pomelo application in daemon mode. Now, it you executed `pomelo start --daemon`, then the application will run in background and be out of the control terminal. All the logs will only output to files.

You can also specify the pomelo `env` by option `-e, --env`.

`pomelo --help` for more help information.

## Enhancement for pushScheduler
#### add an optional parameter for `response` and `push`
In version 0.8, it supports a more fine-grained pushScheduler configuration, and a `connector` can be configured to use more than one pushScheduler. we add an optional parameter `userOptions` for `response` and `push`, for `broadcast`, we just extend its orignal parameter `option`. Users can set the parameter any value, and it is used to select a proper pushScheduler for each `response`, `push` and `broadcast` invocation as a `connector` may be configured to use multi-pushScheduler. An example is as follows:

In Handler, pass a `userOptions` to `next`, `useOptions` will be used in `response`:

```

Handler.prototype.handle = function(msg, session, next) {

    // do handling
    // var resp = ...
    // var userOptions = ...

    cb(null, resp, userOptions);
}

```

An example for `push` invocation:

```

var aChannel = channelService.getChannel('aChannel');

// var userOptions = ...

aChannel.pushMessage(route, msg, userOptions, cb);

```

For `broadcast`, the previous versions have supported the parameter `option` which just contains two available fields `binded` and `filterParam`. In 0.8, we just extend it as `userOptions`, `binded` & `filterParam` are already available and stand same semantics, and users can define more fields in the `userOptions`.

For `errorHandler`, its signature in previous versions is:

```
var errorHandler = function (err, msg, resp, session, cb) {
  // ...
}
```

Now, as adding the parameter `userOptions`, so the new `errorHandler` should be defined as follows:

```

var errorHandler = function (err, msg, resp, session, opts, cb) {
  // ...
}
```

In 0.8, it is compatiable to the previous `errorHandler`, however, the new `errorHandler` is recommended.

### Multi-pushScheudler configuration and selecting
In 0.8, a `connector` can be configured to use more than one pushScheduler, for every `response`, `push` and `broadcast`, it can select a specified pushScheduler based on its rule `selector`, and the optional parameter `userOptions` can be used to calculate the selecting. An example which shows how to do this is as follows:

```

app.configure('development', 'connector', function() {
  app.set('pushSchedulerConfig', {
    scheduler: [ 
      {
        id: 'direct',
        scheduler: pomelo.pushSchedulers.direct
      },
      {
        id: 'buffer5',
        scheduler: pomelo.pushSchedulers.buffer,
        options: {flushInterval: 5000}
      },
      {
        id: 'buffer10',
        scheduler: pomelo.pushSchedulers.buffer,
        options: {flushInterval: 10000}
      }
    ],
    selector: function(reqId, route, msg, recvs, opts, cb) {
       // opts.userOptions is passed by response/push/broadcast
       console.log('user options is: ', opts.userOptions);
       if(opts.type === 'push') {
         cb('buffer5');
         return;
       }
       if (opts.type === 'response') {
         cb('direct');
         return ;
       }
       if (opts.type === 'broadcast') {
         cb('buffer10');
         return ;
       }
    }
  });

```
The change for configuration of pushScheduler also keeps compatiable to previous versions.

## RPC invocation improvement

When initiating a rpc invocation, in 0.8, it can skip the route calculating and dispatch it to a specified server or a kind of servers which have the same serverType. An example is as follows:

```

// route

var routeParam = session;
app.rpc.area.playerRemote.leaveTeam(routeParam, args..., cb);

// to specified server 'area-server-1', 0.8 supports.
app.rpc.area.playerRemote.leaveTeam.toServer('area-server-1', args..., cb);

// broadcast to all the area servers, 0.8 supports.
app.rpc.area.playerRemote.leaveTeam.toServer('*', args..., cb);

```

## Lifecycle callback

Pomelo 0.8 exposes the lifecycle callbacks of application to developers, and this allows developers to do operations in lifecycle of different type of servers. Existing lifecycle callbacks include beforeStartup，afterStartup，beforeShutdown，afterStartAll. 

###beforeStartup(app, cb)
before application start components callback
####Arguments
+ app - application object
+ cb - callback function

###afterStartup(app, cb)
after application start components callback
####Arguments
+ app - application object
+ cb - callback function

###beforeShutdown(app, cb)
before application stop components callback
####Arguments
+ app - application object
+ cb - callback function

###afterStartAll(app)
after all applications started callback
####Arguments
+ app - application object

usage: add lifecycle.js in game-server/app/servers/sometype server/, the example is as follows:

```

module.exports.beforeStartup = function(app, cb) {
	// do some operations before application start up
	cb();
};


module.exports.afterStartup = function(app, cb) {
	// do some operations after application start up
	cb();
};

module.exports.beforeShutdown = function(app, cb) {
	// do some operations before application shutdown down
	cb();
};

module.exports.afterStartAll = function(app) {
	// do some operations after all applications start up
};


```

##Expose rpc filter interface

On the advice of the netizens, pomelo exposes rpc filter interfaces including rpcBefore、rpcAfter、rpcFilter. The usage is similar to handler, and the example configuration is as follows:

```

app.configure('production|development', function() {
	// configurations
	app.filter(pomelo.filters.time());
	app.rpcFilter(pomelo.rpcFilters.rpcLog());
});

```

##Simplify configuration 
0.8 pomelo simplifies the configuration of servers.json by adding the clusterCount field, and this makes pomelo more suitable for deployment of large-scale application.

origin configuration:

```

     "connector":[
             {"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort": 3050, "frontend": true},
             {"id":"connector-server-2", "host":"127.0.0.1", "port":4051, "clientPort": 3051, "frontend": true},
             {"id":"connector-server-3", "host":"127.0.0.1", "port":4052, "clientPort": 3052, "frontend": true}
         ],
        "chat":[
             {"id":"chat-server-1", "host":"127.0.0.1", "port":6050},
             {"id":"chat-server-2", "host":"127.0.0.1", "port":6051},
             {"id":"chat-server-3", "host":"127.0.0.1", "port":6052}
        ],
        "gate":[
           {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true}
        ]
```

0.8 pomelo configuration:

```

    "connector":[
             {"host":"127.0.0.1", "port":"4050++", "clientPort": "3050++", "frontend": true, "clusterCount": 3}
         ],
        "chat":[
             {"host":"127.0.0.1", "port":"6050++", "clusterCount": 3}
        ],
        "gate":[
           {"host": "127.0.0.1", "clientPort": 3014, "frontend": true, "clusterCount": 1}
        ]

```

When using [pomelo-cli](https://github.com/NetEase/pomelo-cli) to add servers dynamically, 'add host=127.0.0.1 port=9000++ serverType=chat clusterCount=3' this command can work as well.

##Pomelo-logger supports dynamic log level

[pomelo-logger](https://github.com/NetEase/pomelo-logger) 0.1.2 supports dynamic log level, and developers can add reloadSecs in config/log4js.json. The parameter means the time checking whether the log configuration has been changed. If the log configuration changed, the levels will be reloaded. The example configuration is as follows:


```

{

  ......

  "levels": {

    "pomelo" : "INFO",
    "rpc-log" : "INFO", 
    "forward-log": "ERROR",
    "con-log": "INFO"
  }, 

  "replaceConsole": true,

  "reloadSecs": 60 * 3

}

```

The above configuration means checking log configuration every 3 minutes.

## Safety
### RPC IP white list

Pomelo 0.8 supports white list for rpc invoking.

###### Principle
Rpc server side will check client ip according to the white list function which provided by developers when a new connection comes. If the client ip is not in the white list, the corresponding socket will disconnect.


###### Usage

If you want to use this function, you only need to pass a white list function (`whitelist: rpcWhitelist.whitelistFunc`) in `remoteConfig`. The example code is as follows:


```
./game-server/app/util/whitelist.js
... ...
var self = this;
self.gWhitelist = ['192.168.146.100', '192.168.146.101'];
module.exports.whitelistFunc = function(cb) {
  cb(null, self.gWhitelist);
  };
... ...
```

```
./game-server/app.js
var rpcWhitelist = require('./app/util/whitelist');
... ...
// configure for global
app.configure('production|development', function() {
... ...
  // remote configures
  app.set('remoteConfig', {
    cacheMsg: true
    , interval: 30
    , whitelist: rpcWhitelist.whitelistFunc
  });
... ...
}
```

* Detail information can refer to [lordofpomelo](https://github.com/NetEase/lordofpomelo/tree/dev)


### pomelo-admin ip white list

###### Principle

This function is similar to rpc ip white list.

###### Usage

If you want to use this function, you only need to pass a white list function (`whitelist: adminWhitelist.whitelistFunc`) in `masterConfig`. The example code is as follows:

```
./game-server/app/util/whitelist.js
... ...
var self = this;
self.gWhitelist = ['192.168.146.100', '192.168.146.101'];
module.exports.whitelistFunc = function(cb) {
  cb(null, self.gWhitelist);
};
... ...
```

```
./game-server/app.js
var adminWhitelist = require('./app/util/whitelist');
... ...
// configure for global
app.configure('production|development', function() {
... ...
  app.set('masterConfig', {
    authUser: app.get('adminAuthUser') // auth client function
    , authServer: app.get('adminAuthServerMaster') // auth server function
    , whitelist: adminWhitelist.whitelistFunc
  });
... ...
}
```

* Detail information can refer to [lordofpomelo](https://github.com/NetEase/lordofpomelo/tree/dev)

##Pomelo-data-plugin

###### Principle

The plugin uses [csv module](https://npmjs.org/package/csv) to resolve the csv configuration file, and uses `fs.watchFile` to monitor file changes. When the plugin is loaded in pomelo, it will load all csv files in specific directory and add a watcher for each file.

###### Installation

```
npm install pomelo-data-plugin
```

###### Usage

```
./pomelo-data-plugin-demo/config/data/team.csv
# team name configuration
# team number, team name
id,teamName
5,The Lord of the Rings
6,The Fast and the Furious
```

The lines start with `#` are comments in csv file. The first line of text is column name(`id` is the main column name, which is necessary), and the next lines are concrete data.

```
var dataPlugin = require('pomelo-data-plugin');
... ...
app.configure('production|development', function() {
  ...
  app.use(dataPlugin, {
    watcher: {
      dir: __dirname + '/config/data',
      idx: 'id',
      interval: 3000
    }
  });
  ...
});
... ...
... ...
var teamConf = app.get('dataService').get('team');
... ...
... ...
```

`dir` means the configuration directory which would be mointored; `idx` means csv the main column of configuration files;`interval` means the time interval of `fs.watchFile`, in milliseconds.


* Detail information can refer to [pomelo-data-plugin](https://github.com/NetEase/pomelo-data-plugin)和[pomelo-data-plugin-demo](https://github.com/NetEase/pomelo-data-plugin-demo)