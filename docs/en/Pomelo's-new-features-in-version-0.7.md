# New features in Pomelo 0.7

In pomelo 0.7, we add crontab feature and developers could add or remove crontab dynamically in different servers; On the advice of the developers, we introduce global filter and this allow developers uniformly  to handle requests in frontend servers; In addition, we introduce simple transaction method, which includes conditions and handlers; Others feature includes auto-complete of pomelo-cli.

## Crontab

Developers can define crontabs through configuration file game-server/config/crons.json, also can add or remove crons by commands provided in [pomelo-cli](https://github.com/NetEase/pomelo-cli). Crontab in pomelo aims at servers, for example if you want to add crontab in chat server:

Firstly, you need to add cron directory in game-server/app/servers/chat, then you can put your crontab code like chatCron.js in game-server/app/servers/chat/cron, for example:

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

Then you can add crontab configuration file crons.json in game-server/config/, the example code is as follows:

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

In crontab configuration file, id is the unique identification in crontab, and it can not be duplicated in the same server; time is the time when the crontab is executed, and which is like the time defined in linux crontab. And the definition of its 7 fields are as follows:
<pre style="bgcolor='#dbdbdb'">
*     *     *     *   *    *        command to be executed
-     -     -     -   -    -
|     |     |     |   |    |
|     |     |     |   |    +----- day of week (0 - 6) (Sunday=0)
|     |     |     |   +------- month (1 - 12)
|     |     |     +--------- day of month (1 - 31)
|     |     +----------- hour (0 - 23)
|     +------------- min (0 - 59)
+------------- second (0 - 59)
</pre>

0 30 10 * * * means crontab is executed at 10:30 every day; serverId is an optional field, if you define this field, the crontab will be executed in specified server, if not the crontab will be executed in this type of server; action is the specified method which will be invoked, chatCron.sendMoney means the method sendMoney in game-server/app/servers/chat/cron/chatCron.js will be invoked.

Developers can use addCron or removeCron provided by [pomelo-cli](https://github.com/NetEase/pomelo-cli) dynamically add or remove crontab, and the necessary parameters in addCron includes id,action,time; the necessary parameters in removeCron is just id, serverId and serverType is alternative;

```
addCron id=8 'time=0 30 11 * * *' action=chatCron.sendMoney serverId=chat-server-3

removeCron id=8
```

## Global filter

In previous edition of pomelo, filter only can be used in backend server. On the advice of the developers, we add global filter in frontend server in 0.7 and this allow developers uniformly to handle requests in frontend servers. The previous handle process is: frontend server -> beforeFilter -> backend server -> afterFilter, and the current handle process is: globalBeforeFilter -> frontend server -> beforeFilter -> backend server -> afterFilter -> globalAfterFilter. The error process is like before, errors will be handled in globalErrorHandler which can be defined in application.

The filter can be used as global filter, and the configuration is as follow:

```
app.configure('production|development', function() {
  app.globalFilter(pomelo.serial());
});
```

## Transaction
In pomelo we provide simple transaction functionality. Developers can define condition methods and handler methods, meanwhile they can define retry times if error occurs in handlers. The transaction will execute the condition methods first, if errors occurs in conditions then the transaction will be abolished, if  conditions are all passed successfully then the handlers will be executed, when errors occurs in handlers the transaction will retry handlers based on the retry times, the default value is one. The process of transaction will be recorded in log file, including the error information which developers can handle in program; the corresponding API is as follow:

####transaction(name, conditions, handlers, retry)
transaction method
#####Arguments
+ name - transaction name
+ conditions - transaction conditions
+ handlers - transaction handlers
+ retry - retry times of handlers if error occurs in handlers

the example code is as follow:

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

## Auto-complete of pomelo-cli
In newest edition of [pomelo-cli](https://github.com/NetEase/pomelo-cli), we provide command auto-complete functionality.

## Rename some pomelo terminology
In newest edition of pomelo, we rename some confusable pomelo terminology. The modifications are as follows:

- LocalSession and LocalSessionService rename as BackendSession and BackendSessionService, in your code you need to pay attention:

```javascript

// deprecated
var localSessionService = app.get("localSessionService");

// recommanded 
var backendSessionService = app.get("backendSessionService");

```
- MockLocalSession rename as FrontendSession

- Component scheduler rename as pushScheduler, need to pay attention for configuring pushScheduler:
```
 app.set("pushSchedulerConfig", opts);
```

- Component proxy and remote's option cacheMsg rename as bufferMsg:
```
// deprecated
app.set("proxyConfig", {cacheMsg: true});
app.set("remoteConfig", {cacheMsg: true});

//recommended
app.set("proxyConfig", {bufferMsg: true});
app.set("remoteConfig", {bufferMsg: true});
```
And these modifications are compatible for old versions.