In this document, we will discuss about how the servers communicate with each other through RPC invocation. In pomelo, RPC invocation is completed mainly by proxy component and remote component, and proxy component is responsible for creating RPC client proxy(a.k.a **stub**), allowing developers to use RPC invocation more easily and conveniently; Remote component is mainly responsible for loading the RPC services, including system namespace RPC services which are used inside pomelo framework and user namespace services which are defined by developers. 

The RPC framework of pomelo is mainly to solve the two problems, the first one is the routing policy since there may be more than one server provide the same remote service; the second is the underlying RPC communication. For the first problem, pomelo provides a flexible routing mechanisms, and it allows developers to freely control the required routing information; For the second problem, pomelo now supports two communication protocol, socket.io and native socket. Here we will introduce you the implementation of the RPC client and server.

RPC Client
==============

RPC Client will scan the remotes inside servers to generate client proxy object (a.k.a stub) dynamically at runtime and load routing policy for remote server selection. When user initiates an RPC invocation, RPC client will package the RPC request and send it to a certain remote server based on its routing policy.

### Proxy Component

It is very convenient to initiate an RPC invocation, an example is shown as follows:

```javascript
app.rpc.chat.chatRemote.add (session, uid, serverId, param, cb);
```

Proxy component will create a RPC client at startup and subscribe to master server for server changing events, including adding server, removing server and replacing server. When these events are emitted, proxy component will dynamicly update the cached servers' status according to the corresponding server changing event. For example, when a new server is added, proxy component will generate the proxy object for it and when a server is removed, proxy component will remove the server proxy object. In other word, proxy will know all the remote services in the server cluster. Proxy component will mount all its server proxy object to app.rpc, so developers can initiate an RPC invocation with **app.rpc.<ServerType>.<Remote>.<Method>(<args...>)**.

### RPC Client

Here is the architecture of RPC client, it has several layer abstractions :

<center>
![rpc client](http://pomelo.netease.com/resource/documentImage/rpc-client.png)
</center>

The bottom layer is mail box layer, it hides the underlying communication protocol details. A mail box corresponds to a remote server. Mail box provides a unified interface for upper layers, such as: connect, send, close, etc. The implementation of mail box can be various on the underlying transport protocol, message buffer queue , the data packing style, etc.. Developers can implement their own mail box to meet their own special demands on underlying protocol or buffered mechanism. Pomelo provides two implementations of mail box, with underlying protocol using native socket and socket.io, and the default is socket.io-based mail box.

Upon mail box layer is mail station layer, The mail station layer maintains all the mail box instances for current server. It would forward the RPC message from upper layer to the proper mail box instance by mail box id. Mail station receives a mail box factory function which decides which kind of mail box should be used for a remote server and return the associated mail box instance. It would ask the mail box factory for mail box instance on the first time of current server try to connect to a remote server. So developers could customize the communication mechanism by the mail box factory function.

Route layer is used to provide message routing policies. It recieves a route fucntion and use it to caculate the destination server id with the RPC message and route parameter, and then the id would be passed to the mail station layer mentioned above. Developers can configure the routing policy in app.js by injecting customized routing function to meet their own demands. 

Proxy layer is the top layer, it is used to hide the underlying RPC implenentation details and provide concise RPC invocation interface for developers, which makes the remote method call just like invoking a local method. The only different of the local proxy method from remote service method is that it adds a route parameter used for routing calculation, the router parameter always is session if the RPC is a system namespace RPC in pomelo, developers can use any object as this route parameter if it is an user namespace RPC defined by developers.

### RPC Invoking Flow 

For sending an RPC request, RPC client uses a lazy connecting mechanism, the main idea is that client will not create mail box immediately to connect to remote server when client starts, but it will establish the connection to remote server when the client first initiates an RPC invocation. After client has established, which means it is not the first RPC invocation, the RPC invocation from client to remote server is no longer needed to establish a connection, the RPC request message can be sent directly. Similarly, sending RPC request message here can also be filtered before like client request mentioned before. Pomelo provides several buitin RPCFilter, including RpcLog and toobusy. 

RPC Server
===============

RPC server is responsible for receiving client RPC invocation request and then dispatch it to the appropriate RPC remote services. After the RPC remote service complete to handle the RPC invocation request, RPC server will send the result to RPC client.

### Remote Component
Remote component will create an RPC server when it starts, and load all remote services that the server provides. When stopping remote component, it will close all the RPC services.

### RPC Server

The following is the architecuture of RPC server, being opposite to client architecture, it has several layer abstractions as well:
<center>
![rpc server](http://pomelo.netease.com/resource/documentImage/rpc-server.png)
</center>

The bottom layer is the acceptor layer, it is opposite to mail box layer in client and it is mainly responsible for listening on configured port, receiving RPC request messages and parsing them. So mail box and acceptor must use the same underlying transport protocol and the same message encapsulation format for RPC request messages. If developers want to customize the transport protocol, it is required to implement a mail box for RPC client and a corresponding acceptor for RPC server. Pomelo provides two implemetations of acceptor according to mail box, and the default is socket.io-based acceptor. 

Upon the acceptor layer is dispatch layer. This layer will dispatch the RPC request to proper remote service based on the description information included in RPC request message.

The top layer of the remote service provides actual business logics, which are implemented by developers and automatically loaded by pomelo framework. The code of remote services is placed into the directory **<ServerType>/<Remote>** by default.

### Summary
This document introduces more detail on the communication mechanisms between the RPC client and server, including mail box, mail station, acceptor. Meanwhile, it analyzes the functionalities of RPC-related component proxy and remote of pomelo framework.