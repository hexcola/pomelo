Pomelo has it's own terminology which some may find confusing without a brief explanation. Here we will try and give readers an overview of some common terms you may come across in this tutorial.

Common Terms
=============

### Gate server

In most cases the Gate server does not handle RPC calls. In Pomelo configuration terms, this means it only requires a port that faces toward the client (clientPort) as the Gate server is primarily used for frontend load balancing. Ideally, your Gate server will be the client's first point of contact and serve no other purpose than to assign the client to a connector server. Load balancing strategies can be implemented within the Gate server to assign users to various connector servers.

### Connector server

The Connector server is responsible for receiving incoming connection requests, creating connections with clients, maintaining clients' session information, receiving client requests and forwarding the requests to a specific backend server according to the developers routing policy. When the backend server has process the request or needs to push messages to clients, the Connector server will send messages to clients as an intermediate role. 

The Connector server has a client facing port (clientPort) and an internal port (port). The client port is used for listening to client connections, and port is used for other backend services to communicate.

### Application server

The Gate server and the Connector server are called frontend servers. An Application server is a backend server which performs application logic providing services to clients. An Application server has client requests routed to it via your frontend servers, while interactions between your Application servers will be handled by RPC calls. Logically since your backend servers don't directly communicate with clients, backend servers only require an internal port defined in its configuration.

### Master server

The Master server is responsible for loading configuration files, starting the server cluster through the configuration file and managing all other servers.

### RPC invocation 

Pomelo invokes RPC calls for interprocess communications. These internal RPC calls can be divided into three types:

* Frontend servers forwarding client requests to backend servers
* Backend servers pushing session information requests to frontend servers
* Backend servers pushing messages to frontend servers through channels

In addition to the system RPC calls are your user defined RPC calls. These kind of calls require you to complete the RPC server code.

### Route, Router

A "route" is a unique identifier to a specific service endpoint where clients push messages to your servers, or where clients handle data received from servers. For servers, routes are usually reached with the following route naming convention: <ServerName>.<HandlerName>.<MethodName>, such as "chat.chatHandler.send". In our example, chat is the server name, chatHandler is the handler defined in the chat server and send is a method within the chatHandler.

For the client, its general form will be on[ExpectedEventName] (for our example, onChat). When servers push messages, the client will assign a function to handle the incoming data from the server for display or processing (commonly referred to as a callback). 

Generally speaking, there will be multiple instances of any given application servers running at any given point in time. When a client request arrives a frontend server will dispatch the client's requests to a specific backend server. This is possible due to a distributed routing function router.

The router that can use user's session as well as the contents of its request , does some calculations, it will map it to a specific application server id. The route can be invoked through the application of a type of server to configure their router. If you do not configure it, pomelo will use a default router which uses session routing function inside the uid fields, calculates fields crc32 checksums uid. Note that there is a trap: If the session is not bound with uid, at this time the uid field is undefined, and this may cause all requests to be routed to the same server. So in the actual development developers still need to configure their own router.


### Session, FrontendSession, BackendSession，SessionService，BackendSessionService

In pomelo, one of the most confusing issues is the concept of these three kinds of session and two kinds of service: `SessionService` and `BackendSessionService`. Here is some explanation, which will hopefully make this a lot clearer to you.
Session is an abstraction of client connection , its fields are as follow:

```javasript
{
    id : <session id> // readonly
    frontendId : <frontend server id> // readonly
    uid : <bound uid> // readonly
    settings : <key-value map> // read and write  
    __socket__ : <raw_socket>
    __state__ : <session state>

    // ...
}
```

* Id is the session id, which is unique in global and generated by the increment way;
* FrontendId is the frontend server id which maintains the session;
* Uid is the user id which is binded with the session;
* \_\_socket\_\_ is a reference to the native socket;
* \_\_state\_\_ is used to indicate the current state of session.
* Settings is a key-value map, which is used to keep some custom attributes of session.

From the above analysis, once a session is established, the field id, frontendId, \_\_socket\_\_, \_\_state\_\_, uid are identified, and they should be readonly. The settings should not be freely modified.

Therefore, in the frontend server, we introduce the FrontendSession, which can be seen as a puppet of real session in frontend server, FrontendSession fields as follows:

```javascript
{
    id : <session id> // readonly
    frontendId : <frontend server id> // readonly
    uid : <bound uid> // readonly
    settings : <key-value map> // read and write  
}
```

The following is FrontendSession's function:

* Settings can be set by FrontendSession, and the values of settings in FrontendSession can be synchronized to the real session by invoking the push method of FrontendSession. 
* You can bind uid to the session by invoking the bind method of FrontendSession;
* Of course, the read-only fields in session can also be accessed via FrontendSession, but it can not send the changes to the original session.

BackendSession is similar to FrontendSession, it is used in backend servers and created and maintained by BackendSessionService. 

### Channel

channel can be seen as a container of players, it is used in the cases in which broadcasting is very frequent. When broadcasting to a channel, all the users in the channel will receive the broadcasting message. A player can be contained by multiple channels. Note that channel is server-local, that means application server A and B does not share their channel information.

### Request, response, notify, push

There are four types of messages in pomelo: _request_, _response_, _notify_ and _push_. Client initiates request to server, and then server returns a response after handling the request. Notify message is also sent to server by client, but it does not need a response. Pushing message is sent by server to client actively.

### Filter

Filter can be divided into two categories: _before filter_ and _after filter_. Before filter would do some pre-handling on the request such as recording log for user "login"ing. After filter would do some post-handling on the request, it always does some cleaning up. In after filter, it should not modify the content of the response, because the response has been sent to client before entering into after filters.

### Handler

Handler is used to do business logic, which is located between the before filter and after filter in the request-handling-chain, its signature is declared as follows:

```javascript
handler.methodName = function(msg, session, next) {
  // ...
}
```

Similar meanings to the _before filter_ parameters. Handler processing is complete, if necessary, in the response back to the client, you can return the result packaged as js object passed to the back through the next process.

### Error handler

Error handler is used to handle the exceptions generated from handling requests from clients. In error handler, it can record the exception log, respond error message to clients and so on. Error handler is optional, and it can be registered to pomelo framework using following if necessary:
```javascript
app.set ('errorHandler', handleFunc);
```
The error handler's signature is declared as follows:

```javascript
errorHandler = function(err, msg, resp, session, next) {
  // ...
}
```
### Component

The pomelo framework is composed of a number of loosely coupled components and the pomelo framework can be regarded as a container of component. Each component defines callbacks: start, afterStart, stop.

### Admin client, Monitor, Master

In pomelo administration framework, there are three roles that servers will act as: admin client, monitor, master.

* Monitor, monitor will report its server status to master and respond to the instructions sent by master.

* Master, master server is responsible for collecting all the information of the server cluster and sending instructions to the server cluster.

* Admin Client, is a third-party administration client. It will connect and register to the master and then request the information about the server cluster or send instructions to the server cluster through the master.

### Admin module

Admin module is used for server administration. Each module defines four callbacks and all of them are optional :
* masterHandler(agent, msg, cb), it will be callbacked by master once receiving a request/notify from monitor;
* monitorHandler(agent, msg, cb), it will be callbacked by monitor once receiving a request/notify from master;
* clientHandler(agent, msg, cb), it will be callbacked by master once receiving a request/notify from a third-party client;
* start(cb), it will be callbacked to do some initialization after being loaded by the administration framework.

### Plugin

Plugin is a new extension mechanism and it is added since pomelo 0.6. A plugin is composed of several components and some event handlers to handle the event emitted by the framework. It provides a very flexible mechanism to extend pomelo.

Summary
============

We have briefly described some terms used in pomelo, and these terms will be invoked in the following example. 

Next, we will go to our example distributed chat application. We will [get its source code and install it](Getting-source-code-&-installation) first.
