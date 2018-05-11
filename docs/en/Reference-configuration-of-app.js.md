A project created by pomelo exists two app.js files, one is in game-server directory and the other is in web-server directory.They correspond to the game server and the web server respectively, and each of them is  the start entry of their own server. The following app.js configuration reference is only for the game server.

## Entrance of Application Setting

The app.js is the entrance of pomelo project. When using the pomelo command line to create a new project, a default app.js file will be generated according to the basic project information. The main content is as follows:
```javascript
var pomelo = require('pomelo');
var app = pomelo.createApp();

app.set('name', 'nameofproject');
app.defaultConfiguration();

app.start();
```
Firstly, it creates an application using pomelo; then it sets the application name. After that it makes default configuration of pomelo. These configurations are necessary conditions to create a pomelo project, and finally you can start the application.


## Variable and Property Setting

Application variables can be accessed through set and get methods. For example, to access the server object, the specific code is as follows:
```javascript
app.set('server',server);
var server = app.get('server');
```
Application state information can be opened and closed by enable and disable methods. Furthermore, developers can call enabled and disabled methods to check enabled or disabled. For example, you want to open or close application's filter and check its state, and the specific code is as follows:

```javascript
app.enable('filter');
app.enabled('filter'); //return true

app.disable('filter');
app.disabled('filter'); //return true
```

Developers can load configuration files through the loadConfig method, after loading the parameters will be attached directly to the app object. For example, loading mysql.json file, the specific code is as follows:


```json
{
  "development":
    {
      "host":"127.0.0.1",
      "port":"3306",
      "database":"pomelo" 
    }
}
```

```javascript
app.loadConfig('mysql.json');
var host = app.mysql.host; //return 127.0.0.1
```

## Rules of Configuration

The server's configuration is mainly done by the app.configure method, the parameters of the method are as follows:
```javascript
app.configure([env], [serverType], [function]);
```

The first two parameters are optional, the parameters' descriptions are as follows:
* env: runtime environment, can be set as development, production or development | production.
* serverType: server type, if set this parameter only this type of server do initialization function, otherwise all servers perform initialization function.
* function: the specific initialization operation, it can be any js method.

Some configuration examples are as follows:

### Example One
```javascript
app.configure(function(){
});
```

This configuration brings all servers under all modes (development / production) into effect.

### Example Two
```javascript
app.configure('development', function(){
});
```

This configuration only brings all servers under a fixed mode into effect.

### Example Three
```javascript
app.configure('development', 'chat', function(){
});
```

This configuration only brings chat servers under development mode into effect.

### Initial Content Examples
Developers can make different configurations for different servers according to various requirements of the application. For example, to configure the startup file of mysql in the global:
```javascript
app.configure('development|production', function(){
     app.set('mysql', app.get('dirname')+'/config/mysql.json');
});
```
Furthermore, you can make configuration on a specific server:

```javascript
var initArea = function(){
   //area init
};
app.configure('development|production', 'area', function(){
     initArea();
});
```

## Load Components
Each component has a life cycle, and they are usually loaded during the initialization of application.
Pomelo components include: master, monitor, filter, proxy, handler, remote, server, sync and connection, and their main functions are as follows:

* master: starting the master server.
* monitor: starting each server's monitor service, the service is in charge of collecting information of servers and regularly pushing messages to the master, maintaining the master server's heartbeat connection.
* proxy: generating server RPC client, which is essential for server processes comuniation(except master server).
* handler: loading handlers in front-end servers.
* filter: loading filters for requests service, including filters before and after the calling of rpc.
* remote: loading the back-end servers, and generates rpc server.
* server: starting service of user requests handling in all servers.
* connector: starting session service in front-end servers and receiving user requests.
* sync: starting data synchronization service.
* connection: starting the service of user connection statistical data.

Most of components will be loaded by default, and developers can have custom components depending on the application requirements. Components loading can use the load method, for example:

```javascript
app.load(pomelo.proxy, [options]);  //[options] optional
```

## Router Configuration

The router is mainly responsible for maintaining the routing information, routing calculation, routing result cache, switch routing strategies and update routing information according to the application requirements. Developers can customize different routing rules for different servers, and these can be configured in app.js. For example:

```javascript
app.route('chat', routeUtil.chat);
```

Developers can also customize routing rules for different servers, for example:

```javascript
routeUtil.chat = function(session, msg, app, callback) {
    var chatServers = app.getServersByType('chat'); 
    if (!chatServers) {
     	callback(new Error('can not find chat servers.'));
		return;
    }
    var server = dispatcher.dispatch(session.rid, chatServers);
    callback(null, server.id);
};
```

The callback function returns the id of the server, and here uses the dispatcher on session.rid which just uses the hash processing to do server selection.

## Filter Configuration

When a client request reaches the server, it is processed by filter chain and handlers, and finally generates a response back to the client. The handler implements the business logic, and the filter works for  preparation and rehabilitation to facilitate code modularization and reuse. Pomelo itself provides some filters, and developers can customize filters according to the application requirements.

* serial: ensuring that all requests from the client to the server will be sent in order.
```javascript
app.filter(pomelo.filters.serial());
```
* time: recording requests response time.
```javascript
app.filter(pomelo.filters.time());
```
* timeout: monitoring requests response time, if timeout happens it gives a warning.
```javascript
app.filter(pomelo.filters.timeout());
```

## Complete Reference Sample
```javascript
var pomelo = require('pomelo');
var routeUtil = require('./app/util/routeUtil');
/**
 * Init app for client.
 */
var app = pomelo.createApp();
app.set('name', 'chatofpomelo');
app.defaultConfiguration();

// app configure
app.configure('production|development', function() {
	// route configures
	app.route('chat', routeUtil.chat);
	app.route('connector', routeUtil.connector);
        
        // remote configures
	app.set('remoteConfig', {
		cacheMsg: true, 
		interval: 30
	});

        // filter configures
	app.filter(pomelo.filters.timeout());	
       
       // mysql configures
        app.loadConfig('mysql', app.get('dirname') + '/config/mysql.json');
});

// start app
app.start();

```