# Introduction to servers in lordofpomelo

The server-side of lordofpomelo is a distributed server cluster, including serveral scenes servers, one or more pathfinding servers, chat server , gate server, long connected servers, etc..

![backend server](http://pomelo.netease.com/resource/documentImage/lordofpomelo/servers.png)

### Connector server

Unlike the short connection used in server, in lordofpomelo, all connections between clients and servers are persistence connections. These connections require a certain amount of resources to keep alive. In lordofpomelo, we use websocket protocal to establish persistent connection between html5 game clients and the node.js game servers, the connections servers are used to maintain these connections, and transit message between clients and servers. The connection state is maintained by the form of player session, which is used to the user's login status, user's basic information, as well as the user's websocket connection.

### Gate Server
Gate server is the entrance for websocket clients. For there are always more than one connector servers, a player will never know which one to connect when he first connect to game server. To solve this problem. gate server provide a fixed websocket port for all players to establish a temporary web socket connection. A player will connect to gate server first, after he pass the verification, he will get a connector address provided by servers. Then the client will cut off the connect with gate server, and connect to the connector server.

### Auth server

Auth server is responsible for user authentication. It is the unified entrance of user authentication, and provide RPC interface for other servers. It is used to hide the details of authentication, and provide a uniform authentication interface.

### Scene server

In online game, a game world will divide into several scenes, in consideration of the performance and load balance. In lordofpomelo, a game map is corresponding to a game scene, and sever by a single area server,. A game scene can not be divided. Therefore, the scene is the basic unit of the game world in lordofpomelo. A area server is responsible for maintaining all the entities, and integration of related services.

### Pathfinding server

Pathfinding service is one of the basic services of a game server, it is the foundation of many player's basic action: moving, following with each other and so on.Pathfinding is used to find the shortest path between two points in a game map. It is a typical stateless, computation-intensive calculations. In the lordofpomelo, we separate the pathfinding logic from scene logic, thereby reducing the load of the scene server. Becase the service is stateless,pathfinding server can be easily scale extensions, you just need to add more servers, and they are all the same.
The pathfinding use A* algorithm based on a metric of collisions, provides a general computing interface, and packaged as a independent module(see link).

### Chat server

The chat service is one of the basic services of the online games.In lordofpomelo, chat service is separation from scene logic, and implement by a separate chat server. Chat server maintains all the online player's information, and use these data to communicate with connector servers, to support the communication between players.

### Introduction scene management

The lordofpomelo scene services are used to manage all the entities in the scene, and also used to run most of the game logic. For security and  consistency, all the game logic are running or verifing at server side, and these game logic are running at the area server. So, the scene server is the core server of lordofpomelo, and the most complex server. in the following part, we will analysis the area server:

### lordofpomelo scene management

In lordofpomelo, each game scene corresponding to a separate area server, all the business logics in a scene are running on the server. Cross-scene operations are supported by a global server. The following picture is the functionality inside an area server:

![Area] (http://pomelo.netease.com/resource/documentImage/lordofpomelo/area.png)

#### Entity management

All entities in the game will be loaded into memory when enter the game scene, then all the modifications on these entities will be directly operate in memory, and use data synchronization module to synchronize the changes to the database. This design makes all the data manipulation in game scene into direct memory operations, improving overall performance by avoiding the slow IO operations.

All the definitions of entities in lordofpomelo are in the entity folder of domain directory, using object-oriented design philosophy.All the objects in the scene are inherited from the entity object. The adding and removing operation of entities are implement by a unified interface of area, thereby reducing the cost of objects management.

#### Message Service

The message service in lordofpomelo can be divided into two different types: one-to-one RPC request-response, and one-to-many broadcast messages.
Because the verification logic is running at service side, most of the client's requests need a response from server, these replies use request/response mode like web server.

In lordofpomelo, we use AOI based notification service. i.e: when the server side produce a broadcast message, we will use the AOI service to get a list of players need to notified, and then broadcast messge the list instead of all the players in the area, thereby greatly reducing the number of messages.


#### AOI Service
In a game, player's view is often far less than the size of the scene, and the message they care are what happened in their views. Becase of this, broadcasting all kinds of messages to alll the players in a scene is not necessary and would be very inefficient. So it is the AOI serveice's job to to notified relevant players.

The AOI service in lordofpomelo is based on tower algorithm, we divided the whole map is into a lot of towers, and each tower would maintain a list of objects which in the range of tower. When something happened, we only need to find out which tower we need to notify, and push message to relative object in these towers.