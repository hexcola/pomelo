# The architecture overview of pomelo

This article is about the design and technical choice of pomelo. 
Why we choose node.js, why we use this architecture and why we design in this way. It is based on our experience of game development and a lot of studies on previous game server solutions.

## Why do we choose node.js?
Node.js is astonishingly suitable for game server development.
In the definition of node.js, fast, scalable, realtime, network, all these features are quite suitable for game server. 

Game server is a high density real time network application, the network io advantage makes node.js and game server a perfect match.
But IO is not the only reason for us to choose node.js, here is all the reasons why we choose node.js:
* Network IO and scalability. Perfect match for game servers, which requires realtime, scalability, high density network IO.
* The node.js thread model. Single thread is quite suitable for game server, it helps us to handle all the troubles about concurrency, lock and other annoying questions. Multi-process, single thread is ideal for game server.
* The language advantages. Javascript is powerful, popular and performance good. Further more, if you choose HTML 5 for client, you can reuse a lot of code logic between server and client.

### The game server runtime architecture

A scalable game server runtime architecture must be multi-process, single process does not scale, even node.js. [gritsgame](http://code.google.com/p/gritsgame) from google and [browserquest](https://github.com/mozilla/BrowserQuest) from mozilla all use node.js as game server, but only single process, which means their online users are limited.

### A typical multi-process MMO runtime is following:

 ![runtime architecture of MMO](http://pomelo.netease.com/resource/documentImage/mmoArchitecture.png)

#### Memo of runtime architecture
* Clients connect to the servers through websocket
* Connectors do not handle logic, it just forward the messages to backend server
* The backend servers include area, chat, status and other type servers, they all handle logic. Most of the game related logics are handled in area servers.
* The backend servers will send back the result to connector, connector then broadcast to related clients.
* Master manages all these servers, including startup, monitor, close etc.

### The difference between game servers and web servers
It looks like web servers and game servers are similar, but actually it's not:
* Long connection VS short connection. The game servers must connect with socket, which is critical for realtime network application. Long connection architecture makes it all different, since all the servers are tightly coupled.
* Difference partition strategies. Game server is based on area based partition strategy, because it can minimize cross process invocation. But web servers can be partitioned based on any load balance strategies, which makes web app more scalable.
* Stateful VS stateless. Because of the partition strategy, the game server is stateful, which limits game server's scalability.
* Broadcast VS request/response. Not like web, Game servers need a lot of broadcasts, even a small action must be notified to related players. These broadcasts make network communication a big burden.

### The runtime architecture of a game is so complicated that we need a framework to simplify it.
Not like web, there are not so much open source game frameworks, not event this architecture.
Pomelo is a rescue, it let you write as little code as possible to support this complicated runtime architecture.

## Introduction to pomelo framework
The following is components of pomelo framework:

 ![pomelo framework](http://pomelo.netease.com/resource/documentImage/pomelo-arch.png)

* server management, it is especially important in this multi-process, distributed architecture.
* network, request/response, broadcast, RPC, session, all these construct the game logic flow.
* application, this is crucial for loosely coupled architecture, app DSL, component, context makes pomelo pluggable and easy to extend.

### The design goal of pomelo

* Servers abstraction and extension.

In web app, servers are stateless, loosely coupled, so there is 
no need for a framework to manager all these servers.
Game apps, however, are different. All these servers work tightly together, and there are a lot of server types. We need to support all these server types and servers extension.

* Request/response, broadcast abstraction

We need a request, response mechanism, or more specifically, a request/broadcast mechanism. Since broadcast is the most usual action in game servers, and potentially a performance bottleneck.

* Servers rpc communication

Servers need to talk to each other, although we try to avoid it.We need a rpc framework as simple as possible.

### Introduction to server abstraction and extenstion
#### Servers types
Pomelo divides servers in two types: frontend and backend, here it is:

![server abstractions](http://pomelo.netease.com/resource/documentImage/serverAbstraction.png)

The responsibility of frontend servers :
 * handle client connection
 * maintain session information
 * forward request to backend
 * broadcast messages to clients

The responsibility of backend servers :
 * handle logic, including rpc and frontend logic
 * push result back to fronend

#### The server duck type
Duck type is commonly used in OOP of dynamic language.
Servers, however can also use duck type idea. There are only two types of interfaces for a server:
 * handle client request, we call it handler
 * handle rpc call, we call it remote

All we have to do is to define handler and remote, which can define what the server looks like.

#### The implementation of server abstraction
The simplest way is to corresponding directory structure with server.  

Here is the example:
![directory structure](http://pomelo.netease.com/resource/documentImage/directory.png)

We just design a server called 'area', the behavior of the server is determined by the code in 'handler' and 'remote'.
All we need to do is to fill the code in handler and remote.
To make the server run, we need a small config file called servers.json. Here is the example file:

```json
{
  "development":{
    "connector": [
      {"id": "connector-server-1", "host": "127.0.0.1", "port": 3150, "clientPort":3010, "frontend":true},
      {"id": "connector-server-2", "host": "127.0.0.1", "port": 3151, "clientPort":3011, "frontend":true}
    ],
    "area": [
      {"id": "area-server-1", "host": "127.0.0.1", "port": 3250, "area": 1},
      {"id": "area-server-2", "host": "127.0.0.1", "port": 3251, "area": 2},
      {"id": "area-server-3", "host": "127.0.0.1", "port": 3252, "area": 3}
    ],
    "chat":[
      {"id":"chat-server-1","host":"127.0.0.1","port":3450}
    ]
  }
}
```

### Client request, response, broadcast
Although we use long connection, but the request/response api looks like web. Here is the example:

![request example](http://pomelo.netease.com/resource/documentImage/request.png)

The api looks like ajax request, although it's actually a long connection cross server request. Based on convention over configuration rules, there is no config.
Pomelo also have filter, broadcast and other mechanisms, you make see the details in [pomelo framework reference](https://github.com/NetEase/pomelo/wiki/Pomelo-Framework)

### rpc abstraction
The rpc framework is really simple, it can automatically choose route strategy and route to the target server with no configuration.Here is the picture:

![rpc invocation](http://pomelo.netease.com/resource/documentImage/rpcInterface.png)

The picture above defined a rpc interface in chatRemotejs, the definition is following:
```
chatRemote.kick = function(uid, player, cb) {
}
```

The rpc client just invoke like this:
```
app.rpc.chat.chatRemote.kick(session, uid, player, function(data){
});
```

Notice the session parameter, it's crucial for router. The framework will help you send the message to certain server.


### The pluggable component architecture
Component is pluggable module in pomelo, developers can implement their own component, and just load it. The [framework reference of pomelo](https://github.com/NetEase/pomelo/wiki/Framework-reference-of-pomelo) will discuss it detail. Following are the life cycle of components:
<center>
![components](http://pomelo.netease.com/resource/documentImage/components.png)
</center>

All user have to do is implementing all these interfaces: start, afterStart, stop, and then we can load it in app.js:

```javascript
app.load([name], comp, [opts])
```

## conclusion
The above framework mechanisms construct the base of pomelo framework.  Above these, we can construct libraries and tools, or framework in another abstract level. The following [tutorial](https://github.com/NetEase/pomelo/wiki/Tutorial) will help us use it in real cases. 

 
