## Why chat?
Pomelo is a game server framework, why does the tutorial starts from chat?

Pomelo is really a game server framework, but it is essentially a high real-time, scalable, multi-process application framework. In addition to some special parts of the game library in the library section, the rest of the framework can be used for development of real-time web application. And compared with some node.js real-time application frameworks such as derby, socketstream, meteor etc, it is more scalable.


Because of the complexity of the game in scene management, client animation, they are not suitable entry level application for the pomelo. Chat application is usually the first application which developers contact with node.js, and therefore more suitable for the tutorial.

Generally the beginner chat application of node.js is based on socket.io. Because it is based on single-process node.js, it hit a discount in scalability. For example, if you want to improve it to a multi-channels chat room like irc, the increased number of channels will inevitably lead the single-process node.js overloaded.

## From a single process To multi-process, From socket.io To pomelo
A native chat room application based socket.io, take [uberchat] (http://github.com/joshmarshall/uberchat ) for example.

Its application architecture diagram is as below:

![uberchat](http://pomelo.netease.com/resource/documentImage/uberchat.png)

The server which only contains a single node.js process handle all requests from websocket.

It has following disadvantages:

1. Poor scalability: it only supports single process node.js, can not be distributed according to room/channel, and can not separate broadcast pressure from logic business processing either.

2. Code redundancy: it just makes simple encapsulation of socket.io, and only the server side contains 430 lines of code.

Using pomelo to write this framework can be completely overcome these shortcomings.

The distributed chat application architecture which we want to build is as follow:

 ![multi chat](http://pomelo.netease.com/resource/documentImage/multi_chat.png)

In this architecture, the front-end servers named connector is responsible for holding connections, the chat server is in charge of processing business logic.

Such scale-up architecture has following advantages:

* Load separation: The architecture totally separates the connection code from the business logic code, and this is really necessary especially in broadcast-intensive application like game.Network communication can consume large amount of resource, however, the business logic processing  ability will not be influenced by broadcasting because of the separated architecture.

* Simplify channel switch: Users can switch channels or rooms without reconnecting websocket because of this separated architecture.

* Scale better: We can launch more connector processes to deal with the increase of users, and use hash algorithm to map channels to different servers.

We will start to build this application with pomelo, and you will be amazed to find that it only takes less than 100 lines of code to build such a complex architecture.

## Initialization of Code Structure
Application of the code structure can be initialized by using the following command:

>pomelo init chatofpomelo

The code structure is shown below:

![chat directory](http://pomelo.netease.com/resource/documentImage/chatDir.png)

Description:

* game-server: all the game business logic code is in this directory.The file app.js is the entrance of the server, and all the game logic and functions start from here.

* web-server: web server which used by game server(including login logic), the client js, css and static resources.

* config: Generally, a project needs a lot of configurations and you can use JSON files here. In game project, some configurations have been created such as log, master-server and other servers at initialization. Also, you can add database, map and numerical tabular configuration etc.

* logs: Logs are essential for the project and contain a lot of information which you can get the project runing state from.

* shared: Both some configurations and code resources can be shared between front end and back end if you choose javascript as client language.

Initialization && Test:

install npm package

>sh npm-install.sh

start game server

>cd game-server && pomelo start

start web server

>cd web-server && node app.js

The web server contains the pomelo client code, when the web server is started, the client is automatically loaded into the browser. Clients send requests to the game server via websocket, the server can push messages to the client after connected by pomelo.

Visit http://localhost:3001, if the web server is running successfully, the following page will appear in  browser:

![welcome page](http://pomelo.netease.com/resource/documentImage/welcome.png)

Click the Test Game Server button, the pop of 'game server is ok.' verify the success of the client and server communication.

## Let's start Chat Logic Coding

The logic of chat includes the following sections:

*   Entering: this part of the logic is responsible for registering user information to session, and add user into the channel of chat room.

*   Chatting: this section includes sending requests from the client, and receiving requests by the server.

*   Broadcasting: all clients in the same chat room will receive messages and show messages in browser.

*   Leaving: this section needs to do some clean-up work, including cleaning up the session and channel information.

### Entering

Entering page is as shown below:
User enters user name, name of chat room, user joins the chat room.

![login](http://pomelo.netease.com/resource/documentImage/login.png)

#### Client

Clients need to send a request to the server, the first request must route to the connector process, because the server needs to register the session information for the first time(page code layout in this tutorial omitted).

```javascript
pomelo.request('connector.entryHandler.enter', {username: username}, function(){
}); 
```

The above request string 'connector.entryHandler.enter' representing the name of server type, the file name of the service and the corresponding method name respectively.

#### Server

The connector can receive messages without any configuration, all you have to do is creating a new file named entryHandler.js under the connector/handler directory. We need to implement the enter method, and the server will automatically invoke the corresponding handler, the specific code is as follows:

```javascript
handler.enter = function(request, session, next) {
	session.bind(uid);
	session.on('closed', onUserLeave.bind(null, this.app));
};
```

#### Server add user into channel

Using the rpc method to add the logged in user into the channel.

```javascript
app.rpc.chat.chatRemote.add(session, uid, app.get('serverId'), function(data){});
```

app is the application object of pomelo, app.rpc represents the remote rpc call between the front and the end servers, the last three parameters correspond to the server name, the file name and the name of the method respectively. In order to finish this rpc call, you only need to create a new file named chatRemote.js in chat/remote directory, and implement the add method.

```javascript
handler.add = function(uid, sid, cb){
    var channel = channelService.getChannel('pomelo', true); 
    if(!!channel)
        channel.add(uid, sid);
};
```

In the add method, firstly get the channel from channelService provided by pomelo, then add the user into the channel. This completes a full rpc call, in pomelo it is that easy to finish complex rpc call.

#### User initiate chatting and server receive message

client code:

```javascript
pomelo.request('chat.chatHandler.send', {content: msg, from: username, target: target}, function(){});
```

server code:

```javascript
handler.send = function(request, session, next) {
 var param = {route: 'onChat', msg: msg.content, from: msg.from, target: msg.target};
 // send messages
};
```

#### Server broadcast message and client receive

In server side, add the code into the send method:

```javascript
var channel = channelService.getChannel('pomelo', false);
channel.pushMessage(param);
```

In client side, all users in the same channel receive and show messages.

```javascript
pomelo.on('onChat', function() {
   addMessage(data.from, data.target, data.msg);
   $("#chatHistory").show();
};
```

### Leaving

When user's session is disconnected, remove the user from the channel.

```javascript
app.rpc.chat.chatRemote.kick(session, session.uid, app.get('serverId'), 'pomelo', null);
```

Like entering into the chat room, add the kick method in chatRemote can achieve the functionality of user exits chat room.

```javascript
handler.kick = function(uid, sid, name){
    var channel = channelService.getChannel(name, false);
    if (!!channel) {
        channel.leave(uid,sid);
    }
};
```

## Start
### Configure servers.json
the specific configuration is as follows：
```json
{
  "development":{
       "connector":[
            { "id":"connector-server-1", "host":"127.0.0.1", "port":3050 }
        ],
       "chat":[
            { "id":"chat-server-1", "host":"127.0.0.1", "port":6050 }
        ]
   },
  "production":{
       "connector":[
            { "id":"connector-server-1", "host":"127.0.0.1", "port":3050 }
        ],
       "chat":[
            { "id":"chat-server-1", "host":"127.0.0.1", "port":6050 }
        ]
   }
}
```
The number of servers that developers can determine based on the number of users, which only need to add a line of code including server id, server type, host and port number in corresponding position in the configuration file.

### Run
>pomelo start

##  scale up
Next we can see the scale up in pomelo is so easy.

The architecture diagram now is below:

![single chat](http://pomelo.netease.com/resource/documentImage/single_chat.png)

If we want to scale up, we just need to modify servers.json.
```json
{
  "development":{
       "gate":[
            { "id":"gate-server-1", "host":"127.0.0.1", "port":3014 }
        ],
       "connector":[
            { "id":"connector-server-1", "host":"127.0.0.1", "port":3050 },
            { "id":"connector-server-2", "host":"127.0.0.1", "port":3051 },
            { "id":"connector-server-3", "host":"127.0.0.1", "port":3052 }
        ],
       "chat":[
            { "id":"chat-server-1", "host":"127.0.0.1", "port":6050 }, 
            { "id":"chat-server-2", "host":"127.0.0.1", "port":6051 },
            { "id":"chat-server-3", "host":"127.0.0.1", "port":6052 }
        ]
   },
 "production":{
       "gate":[
            { "id":"gate-server-1", "host":"127.0.0.1", "port":3014 }
        ],
       "connector":[
            { "id":"connector-server-1", "host":"127.0.0.1", "port":3050 },
            { "id":"connector-server-2", "host":"127.0.0.1", "port":3051 },
            { "id":"connector-server-3", "host":"127.0.0.1", "port":3052 }
        ],
       "chat":[
            { "id":"chat-server-1", "host":"127.0.0.1", "port":6050 }, 
            { "id":"chat-server-2", "host":"127.0.0.1", "port":6051 },
            { "id":"chat-server-3", "host":"127.0.0.1", "port":6052 }
        ]
   }
}
```

This way we easily change the single connector, chat server into multiple connector, chat server architecture. After scale up, there are more than one connector server in front, in order to balance the load of different connector servers, we add a gate server. The gate server is mainly responsible for allocate users in different connector servers, in this application we use the user name's hash value to select connector server.

The architecture diagram after scale up is as follow:

![multi chat](http://pomelo.netease.com/resource/documentImage/multi_chat.png)

### Configure router
When extended to multiple servers, we need to add different routing configurations for different types of servers.The code below is the chat server routing configuration, in order to reduce application's complexity, we just do hash processing on room name, the detailed description of the configuration can refer to[Reference configuration of app.js](https://github.com/NetEase/pomelo/wiki/Reference-configuration-of-app.js)。Specific code is as follow：

```javascript
//routeUtil.js
exp.chat = function(session, msg, app, cb) {
    var chatServers = app.getServersByType('chat'); 
    if (!chatServers) {
     	cb(new Error('can not find chat servers.'));
		return;
    }
    var res = dispatcher.dispatch(session.get('rid'), chatServers);
    cb(null, res.id);
};
```
```javascript
//app.js
app.configure('production|development', function() {
       app.route('chat', routeUtil.chat);
});
```

# Source Code
The tutorial source code can be obtained by using the following command:
>git clone https://github.com/NetEase/chatofpomelo.git