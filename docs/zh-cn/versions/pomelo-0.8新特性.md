## pomelo-cocos2d-jsb
对 cocos2d-x javaScript binding 环境有了支持，开发者可以很方便的使用 javaScript 来完成前后端的开发，并发布到   ios,android,web平台上    
详细请看 [pomelo-cocos2d-jsb](https://github.com/NetEase/pomelo-cocos2d-jsb)
## pomelo命令行重构

### 执行pomelo命令时，去除了对特定目录的依赖。
当在非game-server目录下执行`pomelo start`的时候，需要通过命令行选项指出代码的位置。对于log4js配置文件来说，需要将其日志文件的根目录设置为game-server下的目录。这个目录使用${opts:base}来指定, 因此，如果在其他目录下启动应用服务器，请使用新的log4js配置，在template目录下有相应的示例，如下：

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

对于其他的pomelo子命令，如`pomelo list`， `pomelo add`， `pomelo stop`等命令的执行也不再依赖目录，在执行时需要指定master服务器的地址及端口号，如果不指定，默认使用127.0.0.1:3005，因此这些命令可以远程执行。当然执行这些命令的时候，需要相应的权限控制，默认的用户名和密码均为admin，用户可以根据具体需求进行修改。

对于`pomelo masterha`命令，同`pomelo start`类似，也需要指出要执行代码的位置。这些修改保持与前面版本的兼容。

### 对`pomelo start`的改进
当以daemon方式启动pomelo应用的时候，去除了对第三方模块forever的依赖，此时启动的所有应用进程将脱离控制终端在后台运行，要查看所有的启动以及运行日志，只能通过相应的日志文件。

对于`pomelo start`的env，可以支持由用户来自由配置，使用 -e|--env 选项。

使用`pomelo --help` 可以获得更详细的pomelo命令行支持的选项。

## pushScheduler功能增强
### 对response，push增加了一个可选参数
在新版中，支持了更细粒度的pushScheduler选择。对respone，push增加了一个可选参数userOption; 对broadcast，扩展了原有的option参数的使用。用户可以把它定义为任意值，这个参数将会被用来选择具体的pushScheduler以及在schedule调用中使用，每一次的response，push以及broadcast都可以选择使用不同的pushScheduler。具体示例如下：

在Handler中，可以传入一个userOption参数，如下：

```

Handler.prototype.handle = function(msg, session, next) {
    // do handling
    // var resp = ...
    // var userOption = ...
    cb(null, resp, userOption);
}
```

对于push操作，示例如下:

```

var aChannel = channelService.getChannel('aChannel');
// var userOptions = ...
aChannel.pushMessage(route, msg, userOptions, cb);

```

对于broadcast操作，以前的版本已经支持了option配置，具体的配置项仅仅支持了binded和filterParam，新版本中对此进行了扩展，这两个选项继续有效，而且用户可以自定义更多新的选项，使用方式不变。

对于errorHandler， 以前版本的errorHandler为：

```
var errorHandler = function (err, msg, resp, session, cb) {
  // ...
}
```

由于增加了userOption选项，故相应的errorHandler也多了一个参数，如下：

```
var errorHandler = function (err, msg, resp, session, opts, cb) {
  // ...
}
```
这个修改对以前的版本包持兼容，建议使用新的函数签名。

### 多个pushScheduler的配置及选择
在新版中，一个connector可以配置多个pushScheduler，并根据自定义规则selector来进行选择，response以及push新增加的可选参数userOptions可以用于selector的选择计算。下面的示例中就定义了三个不同的pushScheduler，并应用于不同的情况：


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

新的pushScheduler配置与原有的一个connector仅仅支持一个pushScheduler的配置方式保持兼容。

## rpc调用方式改进

 当进行rpc调用的时候，增加了跳过路由计算而直接将调用发送到一个具体的服务器或者广播到一类服务器的调用方式，代码示例如下：

```

// route
var routeParam = session;
app.rpc.area.playerRemote.leaveTeam(routeParam, args..., cb);

// to specified server 'area-server-1'
app.rpc.area.playerRemote.leaveTeam.toServer('area-server-1', args..., cb);

// broadcast to all the area servers
app.rpc.area.playerRemote.leaveTeam.toServer('*', args..., cb);

```

##生命周期回调
在pomelo 0.8版中增加对外提供application的生命周期回调，这样能够让开发者在不同类型的服务器生命周期中进行具体操作。提供的生命周期回调函数包括：beforeStartup，afterStartup，beforeShutdown，afterStartAll。其具体的功能说明如下：

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


具体使用方法：在game-server/app/servers/某一类型服务器/  目录下添加lifecycle.js文件，具体文件内容如下：

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


##rpc filter提供对外接口
根据网友的建议，在新版本中对外提供了添加rpc filter的接口，包括rpcBefore、rpcAfter、rpcFilter，其功能分别为添加before filter, 添加after filter，同时添加两种filter；使用方法与handler的filter类似。在早期版本中提供的enableRpcLog选项，可以通过添加rpc filter代替。

```

app.configure('production|development', function() {
		
	// configurations

	app.filter(pomelo.filters.time());
	app.rpcFilter(pomelo.rpcFilters.rpcLog());
});

```



##简化配置文件
在0.8版的pomelo中对配置文件servers.json进行了精简，通过增加clusterCount字段将原有的配置进行了简化，这样将更加适合大规模应用的部署和运维管理。

原有的配置：

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

0.8版本pomelo支持的简化配置：

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

同时在采用pomelo-cli进行动态增加服务器的时候，同样可以使用add host=127.0.0.1 port=9000++ serverType=chat clusterCount=3 这样的形式同时增加多台服务器。

##pomelo-logger支持动态日志级别
在[pomelo-logger](https://github.com/NetEase/pomelo-logger)0.1.2中，增加了动态改变日志级别的功能，开发者可以在原有的config/log4js.json中配置reloadSecs参数，该参数表示定期检查配置文件是否有更新，如果有更新则重新装载并根据配置文件，更改日志级别。log4js.json配置如下：

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
该配置表示每3分钟检查一次配置文件是否有更新。


##安全性方面

### RPC调用的IP白名单

该功能可以为各个服务器的RPC调用提供IP白名单过滤功能.

###### 原理

RPC服务端每接受一个连接都会抛出一个连接事件, 这个事件中含有该连接的socket.id和RPC客户端IP. RPC服务端会捕获该连接事件, 并调用用户传入的获取IP白名单的函数, 如果该RPC客户端IP不在白名单中, 则立刻将对应的socket断开. 以此来实现RPC调用白名单过滤功能.

###### 使用

使用时只需要向`remoteConfig`的配置中传入一个获取IP白名单的函数(`whitelist: rpcWhitelist.whitelistFunc`)即可, 这个函数需要接受一个回调函数作为其参数, 该回调函数形如`function(err, tmpList) {...}`. 在获取IP白名单的函数内, 拿到IP白名单时(该白名单应为一维`JS Array`), 以类似于`cb(null, self.gWhitelist)`的形式调用IP过滤回调函数.

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

* 具体请参考[lordofpomelo](https://github.com/NetEase/lordofpomelo/tree/dev)

### pomelo-admin的IP白名单

该功能可以为master服务器的admin提供IP白名单过滤功能.

###### 原理

admin服务端每接受一个连接都会抛出一个连接事件, 这个事件中含有该连接的socket.id和admin客户端IP. admin服务端会捕获该连接事件, 并调用用户传入的获取IP白名单的函数, 如果该admin客户端IP不在白名单中, 则立刻将对应的socket断开. 以此来实现master服务器的admin白名单过滤功能.

###### 使用

使用时只需要向`masterConfig`的配置中传入一个获取IP白名单的函数(`whitelist: adminWhitelist.whitelistFunc`)即可, 这个函数需要接受一个回调函数作为其参数, 该回调函数形如`function(err, tmpList) {...}`. 在获取IP白名单的函数内, 拿到IP白名单时(该白名单应为一维`JS Array`), 以类似于`cb(null, self.gWhitelist)`的形式调用IP过滤回调函数.

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

* 具体请参考[lordofpomelo](https://github.com/NetEase/lordofpomelo/tree/dev)


## csv配置文件自动热加载插件pomelo-data-plugin

该插件可以监控指定目录下的所有csv格式的配置文件, 并在某个文件被改变时自动地将其热加载进入Pomelo. 

###### 原理

该插件主要使用[csv模块](https://npmjs.org/package/csv)来解析csv配置文件, 使用`fs.watchFile`函数来监控文件变化事件. 当Pomelo框架启动该插件中的组件时, 组件会加载给定文件夹中的所有csv配置文件, 并为每个文件加一个watcher. 以此来实现csv配置文件自动热加载的功能.

###### 安装

```
npm install pomelo-data-plugin
```

###### 使用

```
./pomelo-data-plugin-demo/config/data/team.csv
# 队伍名称配置表
# 队名编号, 队名
id,teamName
5,The Lord of the Rings
6,The Fast and the Furious
```

上面的csv配置文件中, 以`#`开头的行是注释语句, 正文的第一行应为列名(`id`为该文件的主键列名, 为必须列), 下面的行是对应列名的具体数据.

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

上面代码中的`dir`即为需要监控的配置文件夹; `idx`为`所有`csv配置文件的主键列名(如:`team.csv`所示的`id`); `interval`为`fs.watchFile`函数测试其所监控文件改变的时间间隔, 单位为毫秒. 

* 具体请参考[pomelo-data-plugin](https://github.com/NetEase/pomelo-data-plugin)和[pomelo-data-plugin-demo](https://github.com/NetEase/pomelo-data-plugin-demo)