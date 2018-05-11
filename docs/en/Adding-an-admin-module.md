A pomelo application is usually supported by a server cluster, so administrating the server cluster is very important. 

Pomelo provides a server administration framework, in which there are three roles servers would act: monitor, master and client. 

Master server is responsible for collecting all the information of the server cluster and sending instructions to the server cluster, while monitor will report its server status to master and respond to the instructions sent by master. Client is a third-party administration client, it will connect and register to master and then request the information about the server cluster or send instructions to the server cluster through the master. 

For different applications, the requirement of administration varies.  Therefore, pomelo provides an administration framework, but not a fixed module. Developers can define their own admin-modules and then register them into administration framework to meet their special requirement on administration of the server cluster.

An admin module is composed of several related callbacks, including `monitorHandler`, `masterHandler`, `clientHandler` and `start`, and either of them can be ommited if possible. `monitorHandler` will be callbacked by monitor once receiving a request/notify from master, while `masterHandler` will be callbacked by master once reveiving a request/notify from monitor. And `clientHandler` will also be callbacked by master when master receiving a request/notify from a third-party client. `start` can be used to do some initilization after an admin-module being registered into the administration framework.

To demonstrate how to customize and use admin module, we will implement a admin module in our example chat, namely "timeReport", which will let monitors report their own current local time to master every five seconds. Nevertheless, this functionality has no much practical significance, even "superfluous", but it is reasonable to keep the example simple. Actually, it is a common way to make monitors report something to master, not only local time, but also anything you want. 

Using in Chat
=============

The sample code is in the branch `tutorial-admin-module`, use the following command to switch:

    $ git checkout tutorial-admin-module

* First, we create a file modules/timeReport.js in the game-server/app directory, in which defining monitorHandler, masterHandler and clientHandler(start ommited), the code is shown as follows:

```javascript

module.exports = function(opts) {
  return new Module(opts);
}

var moduleId = "timeReport";
module.exports.moduleId = moduleId;

var Module = function(opts) {
  this.app = opts.app;
  this.type = opts.type || 'pull';
  this.interval = opts.interval || 5;
}

Module.prototype.monitorHandler = function(agent, msg, cb) {
  console.log(this.app.getServerId() + '' + msg);
  var serverId = agent.id;
  var time = new Date(). toString();

  agent.notify(moduleId, {serverId: serverId, time: time});
}

Module.prototype.masterHandler = function(agent, msg) {
  if(! msg) {
    agent.notifyAll(moduleId, testMsg);
    return;
  }

  console.log(msg);
  var timeData = agent.get(moduleId);
  if(! timeData) {
    timeData = {};
    agent.set(moduleId, timeData);
  }
  timeData[msg.serverId] = msg.time;
}


Module.prototype.clientHandler = function(agent, msg, cb) {
  cb(null, agent.get(moduleId));
}

```

* After defining the admin module, we can register it into administration framework of our application by app.registerAdmin, add the following code into app.js :

```javascript

var timeReport = require('./app/modules/timeReport');
app.registerAdmin(timeReport, {app: app});

```

Here app.registerAdmin can accept two or three arguments, if three arguments, the first one must be a string as its moduleId. Otherwise, pomelo will regard the "moduleId" property of the first argument as the moduleId. For our example timeReport, property "moduleId" is defined for timeReport, so we just pass two arguments when calling app.registerAdmin. 

The last argument is options that can be used to configure the admin-module. There are two import options for an admin-module: type and interval. `type` can be `pull` or `push` and it indicates the way how monitor to report information to master, and `interval` indicates reporting-cycle. In our example, timeReport would use the `pull` mode and the reporting-cycle is five seconds by default.



Notes
======

* When register admin-module into administration framework, it may need a moduleId property defined, and it also can be passed as the first argument when calling app.registerAdmin.

* There are two import options for an admin-module: type and interval. `type` indicates the way how to report informations to master and can be `pull` or `push`. If `pull`, master will periodically request the information of monitors,  while in `push` mode,  monitors will periodically report their own information to master. `interval` indicates the reporting-cycle. If `type` is not configured, then the administration action will not be periodical. 

* For masterHandler, it may be callbacked by master in two cases since it uses `pull` mode: one is triggerd every five seconds by a timely `pull` event emitted, another is triggered when master receiving a reported information from monitors. The two cases can be distinguished by the argument `msg`:

    - If it is timely `pull` event, then msg argument is undefined, so at this time, it just call notifyAll to sent request to pull information of monitors. The argument `testMsg` does nothing but showing how to pass argument when sending a request/notify;

    - If it is receiving a reported information reported from a monitor, then argument `msg` will be an object not undefined. At this time, master print the time value to console and cache the value. Of course, this value does nothing but demonstrating. 

    - In practice, the argument `msg` is often used to distinguish the two cases. Considering another case, in which the module uses not pull but push mode, then the monitor will have to distinguish the two cases too, one is the timely `push` event and another is receiving a request/notify from master, and `msg` also works well for this situation. You can have a try to change the `pull` mode to `push` mode.

* For monitorHandler, it implements a very simple logic business. It will be callbacked by monitor when receiving a request/notify from master. It just take the argument `testMsg` brougt by the request/notify and then report its own current local time to master by a notify. In practice, a request/notify can bring more complex argument than `testMsg` here.

* clientHandler will be callbacked by master when there is a third-party administration client sending a request/notify to the master. Here, master just returns the time information it caches. 

Summary
==========

In this section , we introduce how to use server administration framework provided by pomelo and how to customize an admin-module for the framework. In fact, by customizing your own admin-module, you can make monitors report anything to master. For example, in practical applications, a connector server can report information about its logged users to the master server. So far, we have introduced almost all the basic functionalities of pomelo, the following will be a simple [summary for this tutorial](Tutorial-summary " tutorial summary" ).
