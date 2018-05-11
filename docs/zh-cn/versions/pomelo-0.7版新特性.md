在pomelo 0.7版本中，增加了用户自定义定时任务功能，用户能够在不同服务器中动态地增删定时任务；根据网友的建议，在0.7版中增加了全局filter的功能，用户能够在前端服务器对请求进行统一处理；另外新版中增加了事务的机制，提供了一个简单的事务方法，包括条件方法和处理方法；其它新特性还包括[pomelo-cli](https://github.com/NetEase/pomelo-cli)的命令自动补全功能。

##定时任务
用户能够通过配置文件或者[pomelo-cli](https://github.com/NetEase/pomelo-cli)的命令addCron和removeCron对定时任务进行动态调度。
定时任务是针对具体服务器而言，例如需要在chat服务器中配置定时任务：

首先在game-server/app/servers/chat目录下增加cron目录，在game-server/app/servers/chat/cron目录下编写具体的执行的任务的代码chatCron.js，例如：

```
module.exports = function(app) {
  return new Cron(app);
};
var Cron = function(app) {
  this.app = app;
};
var cron = Cron.prototype;

cron.sendMoney = function() {
  console.log('%s server is sending money now!', this.app.serverId);
};
```

然后在game-server/config/目录下增加定时任务配置文件crons.json，具体配置文件如下所示：

```
{
   "development":{
        "chat":[
             {"id":1, "time": "0 30 10 * * *", "action": "chatCron.sendMoney"},
             {"id":2, "serverId":"chat-server-1", "time": "0 30 10 * * *", "action": "chatCron.sendMoney"}
        ]
    },
    "production":{
        "chat":[
             {"id":1, "time": "0 30 10 * * *", "action": "chatCron.sendMoney"},
             {"id":2, "serverId":"chat-server-1", "time": "0 30 10 * * *", "action": "chatCron.sendMoney"}
        ]
  }
}
```

在配置文件crons.json中，id是定时任务在具体服务器的唯一标识，且不能在同一服务器中重复；time是定时任务执行的具体时间，时间的定义跟linux的定时任务类似，一共包括7个字段，每个字段的具体定义如下：
<pre style="bgcolor='#dbdbdb'">
*     *     *     *   *    *        command to be executed
-     -     -     -   -    -
|     |     |     |   |    |
|     |     |     |   |    +----- day of week (0 - 6) (Sunday=0)
|     |     |     |   +------- month (0 - 11)
|     |     |     +--------- day of month (1 - 31)
|     |     +----------- hour (0 - 23)
|     +------------- min (0 - 59)
+------------- second (0 - 59)
</pre>

0 30 10 * * * 这就代表每天10:30执行相应任务；serverId是一个可选字段，如果有写该字段则该任务只在该服务器下执行，如果没有该字段则该定时任务在所有同类服务器中执行；action是具体执行任务方法，chatCron.sendMoney则代表执行game-server/app/servers/chat/cron/chatCron.js中的sendMoney方法。

通过[pomelo-cli](https://github.com/NetEase/pomelo-cli)的addCron和removeCron命令可以动态地增加和删除定时任务，其中addCron的必要参数包括：id,action,time；removeCron的必要参数包括：id；serverId和serverType是两者选其一即可。例如：

```
addCron id=8 'time=0 30 11 * * *' action=chatCron.sendMoney serverId=chat-server-3

removeCron id=8
```

##全局filter

在之前pomelo的版本中，filter是在后端服务器进行消息拦截并进行相应处理；根据网友的意见，在0.7版中在前端服务器增加了filter,请求在前端服务器就可以进行统一处理。请求的处理过程由之前的：前端服务器 -> beforeFilter -> 后端服务器 -> afterFilter 变为：globalBeforeFilter -> 前端服务器 -> beforeFilter -> 后端服务器 -> afterFilter -> globalAfterFilter。同之前的filter的错误处理过程一样，全局filter的错误全部进入globalErrorHandler处理。

全局filter与以前的filter可以相互通用，具体的配置样例如下：
```
app.configure('production|development', function() {
  app.globalFilter(pomelo.serial());
});
```

##Transaction
在新版本中，pomelo提供简单的事务处理的功能；开发者可以设置相应的事务处理条件和实际事务处理的方法，同时开发还可以定义事务处理的重试次数。事务的具体执行过程是先执行开发者定义的事务处理条件，如果条件报错则直接终止整个事务，如果条件执行通过，再开始执行相应的事务处理方法，当在事务处理方法执行的过程中出现错误则根据开发者定义的重试次数进行执行重试，默认重试次数为1；所有的事务处理的结果都会在相应的日志文件中进行详细记录，开发者可以根据错误日志对失败的事务进行相应处理；相应的API如下所示：

####transaction(name, conditions, handlers, retry)
事务处理方法
#####Arguments
+ name - transaction name
+ conditions - transaction conditions
+ handlers - transaction handlers
+ retry - retry times of handlers if error occurs in handlers

具体的使用示例如下：

```
 var conditions = {
        test1: function(cb) {
          console.log('condition1');
          cb();
        },
        test2: function(cb) {
          console.log('condition2');
          cb();
        }
      };
      var handlers = {
        do1: function(cb) {
          console.log('handler1');
          cb();
        },
        do2: function(cb) {
          console.log('handler2');
          cb();
        }
      };
   app.transaction('test', conditions, handlers, 3);
```

##pomelo-cli自动提示
在最新版本的pomelo-cli中提供命令自动补全的功能，具体可以参考[pomelo-cli](https://github.com/NetEase/pomelo-cli)。

##pomelo 部分术语改名

在最新版本中，对pomelo中一些含混并且可能对用户造成误导的命名做了修改，这样可以使得名字含义更明确，以使得开发者能更好地理解，具体修改如下：

- LocalSession以及LocalSessionService改名为BackendSession和BackendSessionService，具体功能不变，代码中需要注意如下：

```javascript

// deprecated
var localSessionService = app.get("localSessionService");

// recommanded 
var backendSessionService = app.get("backendSessionService");

```
- MockLocalSession 修改为 FrontendSession。

- 组件scheduler改名为pushScheduler， 对pushScheduler组件进行配置选项的时候使用：
```
 app.set("pushSchedulerConfig", opts);
```

- 组件proxy以及remote的配置选项 cacheMsg 本版本中改名为bufferMsg，使得含义更明确：
```
// deprecated
app.set("proxyConfig", {cacheMsg: true});
app.set("remoteConfig", {cacheMsg: true});

//recommended
app.set("proxyConfig", {bufferMsg: true});
app.set("remoteConfig", {bufferMsg: true});
```

这些改名对以前的代码保持兼容。