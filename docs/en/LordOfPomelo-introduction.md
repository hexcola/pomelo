LordOfPomelo is a distributed MMORPG game demo based on Pomelo framework.
 
## The main content of LordOfPomelo
LordOfPomelo covers the core elements of mainstream MMORPG: a number of different game scenes, two kinds of career, a variety of types of tasks, many items and weapons, team, and with a variety of monsters and boss battles. Players can move back and forth across multiple virtual scenes, complete the various tasks, improve levels, and interact with other players. 
 
LordOfPomelo adopts an area-based partition strategy, which means one area is partitioned in one process. This design avoids the complex cross-server transactions while providing the server extension support, developers can add new game scene to improve the whole service side load capacity. In order to evaluate the responsiveness of Pomelo server, we adopted real-time game mode in the scene: the player's attack, skill release, pick up and use of items are performed in real time. 
 
The whole game adopts the standard development mode of Pomelo framework, implements the clustering server management, and you can use the support for the linear expansion in LordOfPomelo, adding a new server to improve the overall load capacity. After several rounds of performance testing and optimization, it can achieve the load capacity of 800 people per single scene, at the same time it can ensure a good response time (less than 100 ms). 
 
## LordOfPomelo overall architecture 
    
<img src="http://pomelo.netease.com/resource/documentImage/lordofpomelo/lordofpomelo-all-arch.png" alt="lordofpomelo overall architecture" width="600px"></img>

As shown above, LordOfPomelo includes two types of server: game-server and web-server. Web-server is http-based web server, players can do the registration and login logic through the web-server. After completion of verification, the player will connect to the game-server cluster and enter the actual game scene. Game-server is LordOfPomelo's core server cluster, including a group of front-end websocket servers, and the back-end game logic server cluster, game-server architecture as shown below: 
 
<img src="http://pomelo.netease.com/resource/documentImage/lordofpomelo/game-server.png" alt="game server" width="600px"></img>

The client in the above image can be any supporting websocket client, LordOfPomelo's client is implemented using HTML5, it not only can run on the PC browser, but also other supporting terminals (such as the iPhone, iPad, android devices configured higher). Players on different platforms can do equal and real-time interaction. 
 
As shown above, game-server cluster is divided into two categories: frontend and backend server, frontend server is a group of websocket server cluster, used to deal with message communications between the websoket clients and is responsible for message forwarding, filtering, broadcasting, and other functions. Backend server is mainly used to handle the game logic, including various types of game logic servers. Among them, the area server is the most important game server, it is mainly responsible for the game scene management, game data updating and saving, client request processing, as well as monsters and NPC behavior driving, etc. These functions are realized through collaborative work with other servers. At the same time, the area server has also take a extensible server form, each map corresponding to a separate area server. You can increase the game areas to disperse the pressure of a single server, to improve the overall load. 
 
## LordOfPomelo analysis 

In the following chapters, we will dig deep into LordOfPomelo architecture.

* [Lordofpomelo Installation Guide] (wiki/LordOfPomelo-installation-guide)
* [Lordofpomelo Server Introduction] (wiki/LordOfPomelo-server-introduction)
* [Lordofpomelo Code Organization] (wiki/LordOfPomelo-code-organization)
* [Lordofpomelo Startup Process] (wiki/LordOfPomelo-startup-Process)
* [Install Lordofpomelo on amazon EC2] (wiki/LordofPomelo-install-amazon-ec2)
* Message [Data compression] (Message-compression)
* [Lordofpomelo Snapshots] (http://pomelo.netease.com/demo.html)
* [Play LordOfPomelo online](http://pomelo.netease.com/lordofpomelo/)