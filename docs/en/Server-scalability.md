If there are only a few users to use our application simultaneously, only a single server is enough to support it. But as users increase, a single server may not be able to bear it, it requires us to scale the server out and use mutli-server to provide service.

Multi-server version of the chat application is on the branch `tutorial-multi-server`, you can use the following command to switch to `tutorial-multi-server` branch:

    $ git checkout tutorial-multi-server

Modifying Configuration 
======================

In pomelo, it is very easy to scale server out, it is not required to modify the source code, but add more servers configuration in the servers configuration file. For our chat example, the new server configuration file config/servers.json is shown as below:

```json
{
  "development": {
    "connector": [
      {"id": "connector-server-1", "host": "127.0.0.1", "port": 4050, "clientPort": 3050, "frontend": true},
      {"id": "connector-server-2", "host": "127.0.0.1", "port": 4051, "clientPort": 3051, "frontend": true},
      {"id": "connector-server-3", "host": "127.0.0.1", "port": 4052, "clientPort": 3052, "frontend": true}
    ],
    "chat": [
      {"id": "chat-server-1", "host": "127.0.0.1", "port": 6050},
      {"id": "chat-server-2", "host": "127.0.0.1", "port": 6051},
      {"id": "chat-server-3", "host": "127.0.0.1", "port": 6052}
    ],
    "gate": [
      {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true}
    ]
  },
  "production": {
    "connector": [
      {"id": "connector-server-1", "host": "127.0.0.1", "port": 4050, "clientPort": 3050, "frontend": true},
      {"id": "connector-server-2", "host": "127.0.0.1", "port": 4051, "clientPort": 3051, "frontend": true},
      {"id": "connector-server-3", "host": "127.0.0.1", "port": 4052, "clientPort": 3052, "frontend": true}
    ],
    "chat": [
      {"id": "chat-server-1", "host": "127.0.0.1", "port": 6050},
      {"id": "chat-server-2", "host": "127.0.0.1", "port": 6051},
      {"id": "chat-server-3", "host": "127.0.0.1", "port": 6052}
    ],
    "gate": [
      {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true}
    ]
  }
}

```

Configuring Router
=====================

Comparing with the example before where there is only one server for a certain server type, here we configure more than one servers for server type `connector` and `chat`. Therefore it is needed to consider how to dispatch user requests to multi-server.

For `gate` server, in the previous example, because there is only one `connector` server, so gate will always return this server if requesting. But here there are multiple connector servers, from which gate should select one using some selecting strategy. We use a utility function `dispatch` to select appropriate server, which uses crc32 checksum of uid modulus the number of connector servers as the connector server index, rough code is presented below:

```javascript
// util/dispatcher.js
module.exports.dispatch = function (key, list) {
  var index = Math.abs (crc.crc32 (key))% list.length;
  return list[index];
};

// GateHandler.js
handler.queryEntry = function (msg, session, next) {
  // ...

  // get all connectors
  var connectors = this.app.getServersByType ('connector');

  // ...

  var res = dispatcher.dispatch (uid, connectors); // select a connector from all the connectors
  // do something with res
};

```

When requests come, because there are multiple chat servers now, so it is also required to select one to handle coming requests. In other words, frontend server should select a backend server and route the requests to it. Here we use the previously mentioned utility function dispatch to select appropriate chat server and configure the routing strategy using application.route, shown as follows:

```javascript
// app.js

// routing strategy definition for chat server
var chatRoute = function (session, msg, app, cb) {
  var chatServers = app.getServersByType ('chat');

  if (! chatServers | | chatServers.length === 0) {
    cb (new Error ('can not find chat servers.'));
    return;
  }

  var res = dispatcher.dispatch (session.get ('rid'), chatServers);
  cb (null, res.id);
};

app.configure ('production|development', function() {
  app.route ('chat', chatRoute);
});

```

In fact, routing requests from frontend server to backend server is a RPC invocation. Here `chatRoute` is the router for chat server, it accepts three arguments and returns chat server id it has selected. For the three arguments, the first one is routing parameter, it is used to calculate the target server, and here it uses session; The second one describes information about the RPC invocation, including the invoked server type, service name, and so on; The third parameter is a context variable, here is app. And the fourth is a callback function used to bring the result out. 

Some Notes
============

* By modifying the server's configuration file and configuring the router, we can easily scale the server out. For our example application, if we want to continue to scale our server out, we just need to modify config/servers.json will. 

* If we have considered scalability of the server at the beginning and  implement the router configuration, so later we can just modify the config/servers.json to scale the server out without modifying the source code.

* In the previous example that there is only ONE server for a server type, the router can be omited, that is because if we do not define the router , then pomelo will use a default router to complete the routing . For chat server, as only one chat server, so pomelo always routes all the requests to this server, the router configuration can be ommited in the example before.

* In practice, it is generally required to implement a certain router, instead of using the default provided by pomelo. When imlementing routers, load balancing should be considered.

Summary
========

In this section, we scale the server out. it is so easy and just modify the server configuration file. In the next section, we will try to use the pomelo's [filter mechanism] (Adding-a-filter "adding filter") to improve our chat application continually.
