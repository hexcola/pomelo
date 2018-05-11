#Framework Overview

There are all kinds of task in a game server, such as managing client connections, managing status of the game world, and executing game logic. Each task requires different resources like IO and CPU that is difficult to manage with one process. So generally, game servers break the server into some multiple processes. For example, a game could have a chat process, connection process, and a scene transition process synchronized by a single master process.

Building this from scratch requires a massive amount of time. While there are frameworks that attempt to abstract out these tasks, like BigWorld (quite expensive and complicated), Pomelo is the leading open-source solution!

![Pomelo Architecture](http://pomelo.netease.com/resource/documentImage/pomelo-arch.png)

* _Server Manager Module_: responsible for defining the server types and creating and monitoring all the service processes.
* _Network Module_: RPC and process-to-process communication.
* _Application Module_: configuration and lifecycle management of the associated service processes.

#Frontend vs. Backend Servers

Pomelo classifies servers into two categories: frontend and backend.

![Server types in Pomelo](http://pomelo.netease.com/resource/documentImage/server-type.png)

_Backend_: game logic.

_Frontend_: communication between clients (e.g. browser or mobile device) and backend servers.

As your game grows, you will need to figure out how to distribute your game code across the front and backend servers.

#Processing Client Requests

##Request and response

Messages from clients fall into two categories: *request* and *notification*. The differences between them are:

![Request and notification messages](http://pomelo.netease.com/resource/documentImage/request-and-response.png)

Requests are bidirectional similar to an HTTP GET request:

		pomleo.request('connector.helloHandler.ask', {msg: 'What is your name?'}, function (resp) {
		  // We can get the name from response
		});

Notifications are unidirectional:

		pomelo.notify('connector.helloHandler.sayHi', {msg: 'Hi'});

##Client Message Processing

Message processing is broken into two parts: *handlers* and *filters*.

A handler is responsible for providing game logic. Filters take care of the pre and post jobs, like logging and timeout handling.

![Process the client request](http://pomelo.netease.com/resource/documentImage/request-flow.png)

###Before Filter

Use a before filter for pre-game-logic tasks, like checking the login status of the current player and logging.

Example:

		filter.before = function(msg, session, next) {
			// Do something with msg and session
			next();
		}

- `msg`: the message object from the client.
- `session`: the session object for the client.
- `next`: callback function to trigger the next object in the message process flow.

If you pass an error to the first parameter of the `next()` function, it means some error has happened and we need to stop the message processing flow (e.g. the current player has not logged in yet).

###Handler

Implement your game logic in a message handler.

Example:

		handler.methodName = function(msg, session, next) {
			// Perform some game logic, like send a message to all clients
			next(200, {response: 'Some response'});
		}

To process a request message, the handler function can pass the response object which is a simple json object as the second parameter to the next callback function.

- `msg`: the message object from the client.
- `session`: the session object for the client.
- `next`: callback function to trigger the next object in the message process flow.

If an error occurs, just pass an error object to the first parameter for the next function.

###Error Handler

Use the `'errorHandler'` to process global errors:

		app.set('errorHandler', function (err, msg, resp, session, next) {
			// Manage error here.
			next();
		});

- `err`: the error object passed to `next()` by before filter or handler.
- `msg`: the message from a client that triggered the error.
- `resp`: the response message passed by handler which would be sent to the client.
- `session`: the client session object.
- `next`: called to trigger processing of the next part of the message processing flow.

###After Filter

The after filter is the final processing point of a message and can be used for tasks like releasing the request resouces or recording the processing time of the request. Note that the response message has already been sent to the client by the time we reach an after filter.

		filter.after = function(err, msg, session, resp, next) {
			// Handle
			next();
		}

- `err`: the error object passed to `next()` by before filter or handler (if there is one)
- `msg`: the message from a client that triggered the error.
- `resp`: the response message passed by handler which would be sent to the client.
- `session`: the client session object.
- `next`: called to trigger processing of the next part of the message processing flow.

###Session

Session is a JavaScript `Object` used to persist player status. There are two kinds of sessions in Pomelo: *internal session* and *backendSession*.

Internal session is generated and located in the frontend server which the client connects with directly. It is the place to store player information. An internal session is cloned and forwarded to the backend server along with the client message. Once cloned, it is called a backend session.

Changes to the BackendSession are not propagated to the internal sessions. To propagate backendSession changes to the internal sessions, call the `push` methods of backendSession.

For more information, see the [API DOC](http://pomelo.netease.com/api.html) backendSessionService.

#Channels and Broadcasting

In this section, we will show how servers push/broadcast messages to clients.

##Channel

There are many messages to push in a game server. For example, we might need to communicate when a player moves from A to B in a scene. In this case the server has to push AOI messages to all other players. A channel is a utility to push these messages.

Channels contain sets of player ids. You can add and remove player ids to a channel. When you push a message to a channel, all the players in the channel would receive the exact same message. You can create any number of channels, and customize each to handle different types of messages for various sections of your game.

##Named vs. Anonymous Channels

There are two kinds of channel in Pomelo: *named* and *anonymous*.

Named channel specify a name and are not released automatically. To destroy a named channel, you call `channel.destroy()`. Named channels are subscription based, like a chat service.

The anonymous channel is accessed via `channelService.pushMessageByUids(...)`. Anonymous channel is used when the members of channel are changed frequently or for temporary messages, such as AOI message.

For more information, see the [API DOC](http://pomelo.netease.com/api.html).

Both channel types function similarly under the hood. Messages are grouped and sent to each frontend server and then the frontend servers send messages to their appropriate clients.

![Broadcast by channel](http://pomelo.netease.com/resource/documentImage/channel.png)

#RPC Framework

In this section, we cover interserver communication.

###Usage of RPC

The Pomelo RPC framework is a helpful utility that links server processes together. The following are some points that the Pomelo RPC framework should consider:

*Routing Rules*: decide which processes should receive a message. The rules will be different depending on the message type. These messages may also affect the status of game objects. Fox example, consider a simple move request from a client. This message should be forwarded to the process that manages the current scene. If the player teleports to a new scene, then all the move requests after that should be forward to the new scene process.

*Protocols*: communication protocol (e.g. TCP vs. UDP) between processes may be different in different games as well.

Pomelo RPC framework introduces abstract layers to simplify and resolve the problems above.

##RPC client

![Architecture of RPC client](http://pomelo.netease.com/resource/documentImage/rpc-client.png)

The RPC message in figure is a typed message that constains a description of the RPC request, including the RPC request type, arguments and so on. The session is a collection of status of the player who lauches the RPC request.

* **Mail box layer** - The mail box layer solves the problem of communication protocol. One mail box stands for a remote server and a mail box uses the remote server id as its own id so that it easy to find out the associated mail box instance by the remote server id. All the details of the communication between current server and the remote server are covered by the mail box instance, such as how to establish the connection and what protocol should be used and how to close the connection. It could implement different mail boxes to support different protocol and it is easy to switch the communication protocol since it just need to choose proper kind of mail box in mail box layer.
* **Mail station layer** - The mail station layer maintains all the mail box instances for current process. It would forward the RPC message from upper layer to the proper mail box instance by mail box id. Mail station receives a mail box factory function which decides which kind of mail box should be used for a remote server and return the associated mail box instance. It would ask the mail box factory for mail box instance on the first time of current server try to connect to a remote server. So developers could customize the communication mechanism by the mail box factory function.
* **Route layer** - The route layer is used to provide the routing rules. It recieves a route function and use it to caculate the destination process id with the RPC message and session pass by upper layer. And then the id would pass to the mail station mentioned above.
* **Proxy layer** - The proxy layer provides the local proxy instances which make the remote method call just like invoking a local method and hides all the details of RPC. The only different of the local proxy method from remote method is that it adds a session parameter which including the status of current player in the first parameter slot of the method. Following is a simple example.

Remote service:

```javascript
  remote.echo = function(msg, cb) {
    // …
  };
```

Local proxy：

```javascript
  proxy.echo = function(session, msg, cb) {
    // …
  };
```

There is another approach to invoke the remote call by rpcInvoke function if the destination server id is available directly.

##RPC server

The layers of RPC server as below:

<center>
![rpc server](http://pomelo.netease.com/resource/documentImage/rpc-server.png)
</center>
<center>
  Architecture of RPC server
</center>

* **Acceptor layer** - The acceptor layer exports the remote services by network. It would listen the port, receive and parse the RPC message by the specified protocol. It should be noted that the acceptor should cooperate with the mail box of remote peer, that means they should use the same protocol to make sure they can communicate with each other without problem. Acceptor is also customized by the acceptor factory function. And acceptor would pass the RPC message to the upper layers.
* **Dispatch layer** - The dispatch layer parses the RPC message, exports the message type and RPC arguments and then dispatches the RPC request to the destination remote service.
* **Remote service layer** - The remote service layer implements the service logics which is provided by the game developers and loaded by Pomelo framework automatically.

#Extension of server

In this section, we will discuss how to extend the ability of a server process.

As mentioned above, we create many kinds of server. And each of them has its own abilities. For example, the frontend server has the ability of receiving messages from client while the backend server has the ability of receiving messages forwarded by frontend servers. And then how should we maintain and reuse thess abilities? Further more, how should we extend the abilities of a process in a more flexible and elegant way?

Combination would be an appropriate approach. Pomelo introduces the component system to achieve the goals above.

##Component

###What is component
In Pomelo, a component is a resusable service unit. A component instance provides some kind of service. For example, the handler component loads the handler codes and pass the client message to the requested handler.

An component instance could be registered into a process context(known as app) and the latter would obtain the ability provided by the component instance. Component instances can cooperate with each other by app. For example, a connector component receives a client request and pass it to app and a handler component may fetch it from app later.

The component system model is described as below:

<center>
![component-system](http://pomelo.netease.com/resource/documentImage/component-system.png)
</center>
<center>
  Component system
</center>

In code, component is a simple class that implements some necessary lifecycle interfaces and app would fire the lifecycle callbacks for each component instance during each phase.

<center>
![components](http://pomelo.netease.com/resource/documentImage/components.png)
</center>
<center>
  Lifecycles of component
</center>

* `start(cb)` - Server start lifecycle callback which would be invoke during the process starting stage. NOTICE: Each component has to invoke the ```cb``` function to continue the next steps. The component also could pass a error argument to ```cb``` to denote that current component fails to start which would let app to terminate the process.
* `afterStart(cb)` - Server after start lifecycle callback which would be invoke when all the registed components in current process have started. It gives the components a chance to do some cooperating initialization.
* `stop(force, cb)` - Server stop lifecycle callback which would be invoke when the server process is going to stop. Components can do some clear job, such as flush data to database in this lifecycle. Force argument is true means all the components should stop immediately.

###Abstraction levels of Pomelo

Based on the component system, app fact is the backbone of the process. It loads all the registed components and drives them throughout the lifecycles. But app would not be involved into the details of each components. All the jobs to customize a server process is just picking out the necessary components and composing them into the app. So the app is clear and flexible and the components is highly reusable. Further more, the component system summarizes all the server types into a uniform process finally.

<center>
![components](http://pomelo.netease.com/resource/documentImage/abstract-level.png)
</center>
<center>
  Abstraction level of Pomelo
</center>

###How to register a component

Register a commponent to app as below:

```javascript
app.load([name], comp, [opts])
```

* `name` - optional component name. Named component instance can be accessed by app.components.name after loaded.
* `comp` - component instance or component factory function. If comp is a function, app would take it as factory function and ask it for a component instance. Factory function takes two arguments app and opts(see below) and return a new component instance.
* `opts` - optional argument that would be pass as the second argument of component factory function.

#Summary
We break the whole game server into small services to clean up the architecture and improve the scalability of game server. And then we summarize all the service types into two kinds of server containers frontend and backend server to simplify the model. And we discuss how the message flows from client to server, from server to client and between the servers. At last, we introduce the component system to combine all the pieces above together to form a unified process. Overall Pomelo provides a scalable and flexible framework to support the game server develop and hide all the noisy and complicated jobs from game developers.

Enjoy the game develop and Pomelo!

More information please refer to [API DOC](http://pomelo.netease.com/api.html), [quick start](https://github.com/NetEase/pomelo/wiki/Quick-start-guide) and [architecture overview](https://github.com/NetEase/pomelo/wiki/Architecture-overview-of-pomelo)
