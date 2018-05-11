As we know, the work has been done by a game server based on pomelo is to manage life-cycle of the components game server reloads, which provide all the functionalities of the server. There are more than ten builtin components provided by the framework, and these components are loaded by various servers, and provide various functionalities. Here we will introduce all the builtin components by presenting their functionalities.

### Master 

Master componenent is only loaded by master server, its main functionalities include starting all application servers, manage and monitor all the application servers and respond to the requests from a management client.

In "start" method of the master component, it will spawn all the application servers according to the servers configuration infomation that user supplies. 

After "start" method invocation done, it will listen on configured port to accept connection and registration from the application servers, and then it can be used to collect monitoring information reported by the application servers, send commands to the application servers, and handle requests from management client, such as [pomelo-cli] (Pomelo-cli use), which may request viewing a particular server's status , adding a server, turning off a server, etc.. Taking adding a server as a example, when management client sends a request to master for adding a server , it will offer some server parameters, such as server type, host ip, listening port, etc.. And then, master component will handle this request, it will spawn a new server based on the parameters offered, and then broadcasts the newly added server's information to other servers that have been spawned.

**Configuration Options**: None.

### Monitor 

Monitor component will be loaded by all the servers, of course including the master server as well. Its main functionality is to establish a connection to the master, and does some management and monitoring work together with master component. Master server itself will load monitor component because master server also collects its own monitoring information.

It can be said monitor component is opposite to the master component, Servers will get some commands from master server and handle it via its monitor component. For some periodic monitoring information, pomelo provides two collecting mode, pull mode and push mode. The pull mode requires master server periodically to send the monitoring commands to monitored servers for getting monitoring infomation, while push mode does this by monitored servers periodically report their own monitoring information to master servers initiatively.

**Configuration Options**: None.

### Connector 

Connector component is a heavyweight component, which has dependencies on session component, server component, pushscheduler component and connection component. Connector component can only be loaded by the frontend servers, which is mainly used to manage client connections. Connector component will create underlying transport connector, listen on the clientPort that frontend servers being configured, bind the events related to client request to corresponding handlers. 

When a client establishes a connection or send a request, connector component will accept it and requires session component to maintain its session, connector component also reports it to the connection component the information for statistical use, finally, connector component will deliver the session obtained from session component. 

After finishing the handling of the request by server component, the response will be returned to the connector component and then connector component deliver it to clients. When responding to clients, there is a buffer approach used to increase bandwidth utilization, and this buffer approach depends on pushscheduler component, that means connector component will not deliver the responses to appropriate clients directly, instead, all the response will be delivered to pushscheduler component, and pushscheduler component will deliver the response to clients according to its scheduling policy, it may be bufferred and timely flush or on time.

**Configuration Options**:

* connector: the underlying transport connector used, default is sioconnector;
* useProtobuf: currently supported only if connector option set to use hybridconnector, default is false, set it to true will enable using protobuf to pack message ;
* useDict: currently supported only if connector option set to use hybridconnector, default is false, set to true will enable the dictionary-based routing message compression ;
* useCrypto: currently supported only if connector option set to use hybridconnector, default is false, set to true will enable digital signature functionality while communicating;
* encode/decode: message encoding and decoding methods, if not configured, it will use encode/decode the underlying connector provides.
* transports: currently supported only if connector option set to use sioconnector(socket.io), it can be websocket, xhr-polling and so on. With this option you can choose the desired mode.

Configure connector component by:

    app.set ('connectorConfig', opts);

### Session

Session component is related to connector component and it is only loaded by frontend servers, it provides a component wrapper for sessionService. After loading session component, it will add "sessionService" property to context app, and can be gotten using `app.get ('sessionService')`. It is mainly used to maintain the client's connection, and create and maintain sessions. If making an analogy with classical TCP, then the connection infomation maintained by session corresponds to "socket returned by TCP server accept call". One session for one connection. Session also maintains other info about the connection, such as the user bind to the connection and other application-specific information. A user can login from multiple clients, that means a user can be bind to several sessions. When pushing/responding messages to clients, it must get the session first and do it via the session.

**Configuration Options**:

* singleSession: set to true will enable forbidden on multiple logins for a particular user simultaneously, default is false. 


Configure session component by:

    app.set ('sessionConfig', opts);

### Connection 

Connection component is a simple component relatively, and it is also only loaded by frontend servers, it is a component wrapper for connectionService, which mainly does some statistical work for connections. connector component will report connection information to connection component while accepting a connection or a client goes offline.

**Configuration Options** : None.

### Server 

Server component is a complex component, which is loaded by all the servers except master server. Server component loads and maintains its Handlers and Filters. Server component accepts requests from connector component, then uses its own before-filter-chain to filter the requests and use its corresponding Handler to handle the requests, and finally responds results to connector component. At last use after-filter-chain to do some cleanup.

Of course, for a request to backend server, it will also be accepted by the server component of the frontend server, and frontend server is not able to handle it, so it will forward the request to backend server via RPC invocation. For backend servers, request is not directly from clients but sys RPC invocation invoked by frontend servers, and then use its filter-handler chain to handle the request. After finishing the handling, backend server will respond the result to frontend server as a return of RPC invocation. This RPC invocation is based on builtin "msgRemote" pomelo provides.

While dispatching requests to backend servers, as there are more than one backend servers of the same type, so it needs a rout policy that called router configured  in frontend servers, it can be achived by Application.route call to configure router.

**Configuration Options**: None.

### Pushscheduler 

The functionality of pushscheduler component is relatively simple components, it is only loaded by frontend servers, and it is related to connector component closely. When connector component receives responses or pushing messages from server component, connector does not directly respond/push it to clients, but schedule it to pushscheduler component. Pushscheduler component will complete the final work to send the responses to clients based on its scheduling policy. Pomelo has implemented two scheduling policies, one is without any buffer, sends the responses directly to the clients, and another is with buffer and periodically sends the buffered responses/pushing-messages to clients.

**Configuration Options** :

* scheduler: scheduling strategy, the default is to directly send to clients, user can implement their own scheduling strategy.

Configure pushScheduler component by:

    app.set ('pushSchedulerConfig', opts);

If you want to enable buffered scheduler, you can do this by:

    app.set('pushSchedulerConfig', {scheduler: pomelo.pushSchedulers.buffer, flushInterval: 20});

Here, flushInterval is refresh cycle, default is 20 milliseconds.

### Proxy 
 
Proxy component is a heavyweight component, it is loaded by all the servers except master server. Proxy component will scan specific application server directory, due to the dynamic nature of javascript language, the meta-information about remotes can easily be obtained, and then use them to generate stubs, put them under global context app. When RPC invoking, proxy component will check whether the invocation is valid using the stub information. Meanwhile, proxy will create a RpcClient while launching a RPC, it is responsible for communicating with the remote server, and get results from RPC invocation. The router configuration is required so there are more than one servers that provide same remote services.

**Configuration Options** :
* bufferMsg, set to true will enable RPC invocation cache, that means it will not directly launch a RPC as long as a RPC invocation presented, default is false.
* interval, if bufferMsg is set to true, interval indicates flush cycle. Otherwise, it does nothing.
* mailBoxFactory, it is used to communicate between RPC invoker and invoked, it can be treated as underlying networking transport abstraction. Developers can customize their own mailBoxFactory, default is wsBoxFactory, which uses websocket as its transport. 

In addition, you can enable RPC invocation log by:

    app.enable('rpcDebugLog');

Configure proxy component by:

    app.set('proxyConfig', opts);

### Remote 

Remote component is a component opposite to proxy component, it is loaded by all the servers except master server. It is used to provide RPC remote services. Remote component will load all remote handlers defined in current server, and listen on configured port, then wait for RPC invocation from RPC clients. When a invocation is accepted, it will dispatch the invocation to corresponding loaded remote hander based on information described inside the invocation, and then return result to RPC client. RPC remote-side also supports the filter on invocations, that is similar to request/response that server component does. 

**Configuration Options** :
* BufferMsg, same as proxy component
* Interval, same as proxy component
* acceptorFactory, it is opposite to mailBoxFactory option for proxy component, it is also used to do networking commumication.

Same as proxy component, users can enable rpcDebugLog for remote component too.

Configure remote component by:

    app.set ('remoteConfig', opts);

### Dictionary 

Dictionary component is an optional component that is not loaded by default, it will be loaded only when connector component is configured to enable "useDict" option. This component will traverse all handler route strings that current server provides and routes strings from file config/dictionary.json that are used at client-side, and then assign each route to a unique small integer to compress route when encoding/decoding messages. Client will negotiate the dictionary during handshaking when establishing connection, and after that, it will use small integer rather than the original route string to communicate for information compression.

**Configuration Options** :
* dict, the file that holds client route strings, default is config/dictionary.json.

Configure dictionary component by:

    app.set ('dictionaryConfig', opts);

### Protobuf

Protobuf component is an optional component that is not loaded by default, it will be loaded only when connector component is configured to enable "useProtobuf" option. This component will load the corresponding proto files and do encoding/decoding messages using protobuf. The configuration file it uses is config/serverProtos.json and config/clientProtos.json.

**Configuration Options** : None.

### Channel 

Channel component maintains the channel information that is loaded by all the servers except master server, it is a component wrapper for channelService, you can get channelService by "app.get ('channelService')" after loading channel component. Channel can be treated as a collection of users that have logined in, each user is corresponding to one or more sessions in frontend servers, developers can push message to a channel by channel component, which will push messages to all users contained by the channel. 

**Configuration Options** :
* broadcastFilter, filter for broadcast. when broadcasting, it will filter the messages before sending messages to client in frontend servers. Its function signature is:

    broadcastFilter(session, msg, filterParam);

Here filterParam parameter is passed via call to broadcast in channelService, as follows:

    channelService.broadcast(type, route, {filterParam: param}, cb)

Configure channel component by:
 
    app.set('channelConfig', opts)

### BackendSession

BackendSession component is loaded by all the servers except master server, it is a component wrapper for BackendSessionService, you can get backendSessionService by "app.get ('backendSessionService')" after loading backendSession component. It generates and maintains backendSessions for backend servers, also it can be used to operate raw sessions that maintained by frontend servers by RPC invocation.

**Configuration Options**: None.

