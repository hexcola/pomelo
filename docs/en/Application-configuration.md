This document introduces how to configure the pomelo framework. As we know, we can configure options for each component, load the configuration file, enable/disable pomelo features to configure the pomelo framework, and all these are placed in game-server/app.js. 

app.js File 
==========

app.js is the entrance of project based on pomelo. In app.js, it firstly creates an instance **app** of class Application, and then uses this app as context of the framework. Developers can mount some global variables or load some configuration to this context, developers can also do some other initializing and configuration operations for the app. At last, it starts the project by call app.start(). The main content of a general app.js is shown as follows:

```javascript

var pomelo = require ('pomelo');
var app = pomelo.createApp();

// some configurating operation for app.

app.configure(<env>, <serverType>, function() {
   // ... 
});

app.configure (....);
app.set (...);
app.route (...);

// ...

// start app
app.start ();

```

app.configure() 
========================

Configuration on servers is always by calling app.configure() method, whose signature is presented as follows: 

```javascript
app.configure([env], [serverType], [function]);
```

The parameter env and serverType are optional, all the parameters are explained as follows:
* env: runtime environment, it can be set to development, production or development|production;
* serverType: target server type, if this parameter is set to T, then only servers whose type is T do configuration logic, otherwise, all the servers do configuration logics.
* function: it is required and is used to holds the configuration logic for servers.

Here are some configuration examples :

#### Example I 
```javascript
app.configure (function() {
  // Do some configuration
});
```
This configuration will affect on all the environment(development or production) for all servers, which is equivalent to directly write configuration code in app.js, code examples are as follows:

```javascript

app.configure (function () {
  doSomeConfiguration ();
});

// <==>

doSomeConfiguration(); // equivalent to above `app.configure`

```

#### Example II 

```javascript
app.configure ('development', function() {
  // Do some configuration just for development env only.
});
```

This configuration will affect all the servers that is in development environment. 

#### Example III 
```javascript
app.configure ('development', 'chat', function () {
  // Do some configuration just for development env and chat server only.
});
```
This configuration will only affect the chat servers in development environment.

#### Configuration Logic

Developers can make different configuration for different servers to meet the server's demands. for example, loading mysql configuration for all the servers: 

```javascript
app.configure ('development|production', function() {
  app.loadConfig ('mysql', app.getBase() + '/config/mysql.json');
});
```
It can also do some configuration on specific servers, for example, doing some initialization for area servers:

```javascript
var initArea = function () {
  // area server initialization
};

app.configure('development|production', 'area', function() {
  initArea ();
});

```

It can do any configurations on any servers in any environments. These configurations include setting a variable to context of the framework that can be used anywhere, enable/disable some features of pomelo, setting options for builtin components, configuring the filter for a specific server, etc.. An example is shown as follows:

```javascript
app.configure('development|production', 'chat', function() {
  app.route ('chat', routeUtil.chat); // configure route
});

app.enable('systemMonitor'); // enable feature

app.configure ('development|production', 'gate', function() {
  app.set ('connectorConfig', {
    connector: pomelo.connectors.hybridconnector,
    heartbeat: 3
  }); // configure opts for connector component
}); // configure connector for gate server
```

Accessing Context Variable 
========================

The app instance has getter/setter for context variables:

```javascript
app.set(name, value, [isAttach]);
app.get(name);
```
* For setter, there are three parameters: variable name , variable value and an optional parameter isAttach. If isAttach is set to true, the variable will be attached to app instance as a property of app. The default for isAttach is false if omitted.
* For getter, it is very simple to obtain a variable value by its name.

Example code is presented as follows:

```javascript
app.set('server', server);
var server = app.get ('server');

app.set('service', service, true);
var service = app.service;
```

In pomelo framework, developers can set options for builtin components of pomelo by app.set. Also, the builtin service such as backendSessionService, channelService provided by pomelo framework can be obtained by app.get, example code is presented as follows:

```javascript

app.set ('connectorConfig', {
  // ...
}); // set options for connector component

var backendSessionService = app.get ('backendSessionService'); // get backendSessionService instance
```

Developers can also set some customized variables into app, and access them anywhere by app.get.

Disabling/Enabling Features
=========================

Some features of pomelo can be enabled and disabled. Developer can check status of a feature by enabled and disabled method. For example, to disable and enable rpc debug log feature and check its status, the sample code is shown as follows:

```javascript
app.enable ('rpcDebugLog');
app.enabled ('rpcDebugLog'); // return true
app.disable ('rpcDebugLog');
app.disabled ('rpcDebugLog'); // return true
```

In pomelo framework, when it is required to do more detailed monitoring and management, you can enable systemMonitor feature to load additional Modules, sample code is shown as follows:

```javascript
app.enable ('systemMonitor'); // enable system monitor
```

Developers can also make a feature for pomelo that cant be disabled/enabled by app.disable/app.enable and checked by app.enabled/app.disabled.

Loading Configuration File 
========================
Developers can load configuration file by app.loadConfig, and the configuration information will be mounted directly to app object. For example, loading mysql.json file which should be placed in game-server/config directory, the following is sample code:

```javascript
// mysql.json
{
  "development":
  {
    "host": "127.0.0.1",
    "port": "3306",
    "database": "pomelo"
  }
}

// app.js
app.loadConfig('mysql.json');
var host = app.mysql.host; // return 127.0.0.1

```
Of course, Developers can use app.loadConfig to load any json-formatted configuraton file which is placed in game-server/config directory for any purposes.

Loading Component
===================

The functionalities of pomelo is provided by its loaded components, pomelo would load different builtin components by default based on the type of server, and developers can implement a customized component and then load it to pomelo manually. Loading a component uses app.load, sample code is as follows:

```javascript
app.load(HelloWorldComponent, [opts]); // opts is optional
```

Using Plugin
===========
pomelo can also use a customized plugin, a plugin is composed of multiple components and event handlers for responding to events emitted by application, Using a plugin uses app.use, sample code is as follows:

```javascript
// app.use (<plugin>, <plugin options>);

var statusPlugin = require('pomelo-status-plugin');

app.use(statusPlugin, {
  status: {
    host: '127 .0.0.1 ',
    port: 6379
  }
});
```

Configuring Router
====================

Router is used to calculate the target server for requests from clients. Developers can customize the routing policies for the different servers and then configure it on the servers. The following is an example:

```javascript
// routeUtil.js
app.route('chat', routeUtil.chat);
```

Routing policy can be defined in routerUtil, for example:

```javascript
routeUtil.chat = function(session, msg, app, callback) {
  var chatServers = app.getServersByType('chat');
  if (!chatServers) {
    callback (new Error ('can not find chat servers.'));
    return;
  }
  var server = dispatcher.dispatch (session.rid, chatServers);
  callback (null, server.id);
};
```
In the routing function, The target server id is returned through calling the callback function as an argument. 

Configuring Filter
=====================

When a request from a client arrives at server, it will be handled by filter-chain and handler. Handler is where the business logic works, and it will do some pre and post-handling for business logic in filters. In order to facilitate developers, pomelo provides a number of builtin filter such as serialFilter, timeFilter, timeOutFilter. Developers can customize their own filter according to certain requirement. The following example shows how to configure a filter:

```javascript
app.filter(pomelo.filters.serial()); // configure builtin filter: serialFilter

app.filter(FooFilter); // configure FooFilter as a before & after filter

app.before(beforeFilter); // configure beforeFilter only as a before filter

app.after(afterFilter); // configure afterFilter only as an after filter
```

If the filter is only a before filter, then use app.before method; If the filter is only an after filter, then app.after instead. If the filter is both a before filter and an after filter, app.filter method works.

Registering Module
======================

Pomelo provides monitoring and management framework which can be registered by different Modules for different purposes. Registering a Module uses app.registerAdmin, sample code is shown as follows:

```javascript
  app.registerAdmin (require ('../modules/watchdog'), {app: app, master: true});
```
Developers can also customize their own Module, and then register it to the framework by calling app.registerAdmin.

Configuration File for Servers 
===============================

There are many configuration files placed in game-server/config directory, among them there are two very important configuration file for serves: servers.json and master.json.

Every configuration file includes two environment configurations: development and production. Their configuration item are list as follows:

For master.json:

* id: server id of master server, it is a string ;
* host: host of master indicates where to launch a master server, it can be a domain name or ip;
* port: the listening port for master server, the default is 3005 ;
* args: optional, it is used for node/v8, for example you can configure `"args": "--debug=5858 "` and then you can debug your project.


For servers.json:

* id: server id of application server, it is a string;
* Host: same as master server but for application server here;
* port: listening port for RPC requests;
* frontend: it is a boolean value, for frontend server, it must be set to true. If omitted, the default is false. It should not be configured for backend servers; 
* clientPort: listening port for requests from clients for the frontend servers. It should not be configured for backend servers;
* max-connections: optional, it is used to describe maximum connections that frontend servers can hold. It should not be configured for backend servers too;
* args: optional, same as master server;
* cpu: set cpu affinity for server process, it depends on **taskset** and only works on unix-like platform.

