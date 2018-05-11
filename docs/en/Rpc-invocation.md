 Inter-process communication is indispensable in a multi-process architecture. Pomelo provides an RPC framework that is very friendly to developers. In this section, we will introduce how to initiate an RPC invocation in pomelo application using our chat application as an example. In this example, we implement a "superfluous" time server, which provides time service for other servers. If the gate server received a request, then it will initiate an RPC invocation to time server to get the current time and print it on the console. Nevertheless, this example has no practical significance, it is just a sample to demonstrate how to use the RPC framework.

In fact, there is already RPC invocations used in our chat application before, that is, when a user connects to or leaves from connector server, connector server will initiate an RPC invocation to chat server, and then chat server will handle the user joining and leaving requests. As we have not analyze them in detail, so simply we reimplement a new and simple RPC invocation example, we can also demonstrate you how to add a pomelo application server type at the same time.

Using in Chat
==============

we will add our example chat an RPC invocation and a time server type, the code in the branch `tutorial-rpc` , use the following command to switch:

     $ git checkout tutorial-rpc

* First, we create a time server type, it has a service named timeRemote and the time method is getCurrentTime (arg1, arg2, cb), where arg1, arg2 has no actual meaning, just for demonstrating how to pass arguments while RPC invoking. the file services/time/remote/timeRemote.js is shown as follows:

```javascript

// timeRemote.js
module.exports.getCurrentTime = function(arg1, arg2, cb) {
  console.log("timeRemote - arg1:" + arg1 + ";" + "arg2:" + arg2);
  var d = new Date();
  var hour = d.getHours();
  var min = d.getMinutes();
  var sec = d.getSeconds();
  cb(hour, min, sec);
};

```

Here, in the getCurrentTime, it prints the arg1, arg2 passed from RPC client to console and then gets the current time and return it to RPC client.

At RPC client:

```javascript

// gateHandler.js
var routeParam = Math.floor(Math.random() * 10);
app.rpc.time.timeRemote.getCurrentTime(
  routeParam, arg1, arg2, function(hour, min, sec) {
  //  ...
});

```

The first argument `routeParam` of the RPC service method `getCurrentTime` is used to calculate the route for time servers. `arg1`, `arg2` is just an example for passing arguments from RPC client to RPC server, of course, you can pass any parameters from client to server in an actual RPC invocation. The last argument is a callback which will be invoked when RPC invocation returning, it should have consistent signature with the corresponding service method.

When there are multiple time servers, we should configure the routing strategy for selecting a target time server, namely router. The first argument `routeParam` is used to calculate the route, here it is random number within 0-10. In pomelo, it uses session as routing parameter in many cases, but here we use a different one, the code for configuring the router is as follows:

```javascript

// app.js
var router = function(routeParam, msg, context, cb) {
  var timeServers = app.getServersByType('time');
  var id = timeServers[routeParam% timeServers.length].id;
  cb(null, id);
}

app.route('time', router);

```

Now, adding time server configuration in the servers configuration file config/servers.json as follows:

```javascript

"time": [
  {"id": "time-server-1", "host": "127.0.0.1", "port": 7000},
  {"id": "time-server-2", "host": "127.0.0.1", "port": 7001}
]

```

Well, we have added a time server type to the example chat, and made an example for RPC invocation.

Some Notes
============

* You may have noticed that the definition of timeRemote.js is different from chatRemote.js. In timeRemote.js, we directly export `getCurrentTime` to  `module.exports`, and in chatRemote.js, it use the following way:

```javascript

// chatRemote.js
module.exports = function (app) {
  return new ChatRemote (app);
};

// timeRemote.js
module.exports.getCurrentTime (arg1, arg2, cb) {
  // ...
};

```
In fact both these two way work well, it can export a factory function or an object when implementing a remote service. when loading remote services, if it is a factory function, then pomelo will construct the remote service object using the factory function, otherwise, pomelo will use the server object directly.

* When receiving a request from client, the frontend server will route the request to target backend server by initiating an RPC invocation using the session as RPC routing argument, and this is why the first argument for routing function(router) is always session in many cases. However, in our example, we use a random integer as routing argument, so the first argument of the routing function for time server is the random integer not session. Indeed, the routing argument can be any types.

* The result of RPC invocation is returned through callback, and the callback function can have multiple arguments, that means the RPC invocation can return multiple values. Here, in our example, it returns three values, hour, min, sec.

Summary
==========

In this section, we add to the example chat a time type server and implements an rpc invocation. Next, we'll [ add a component](adding-a-component  "Add Component") to the example chat.
