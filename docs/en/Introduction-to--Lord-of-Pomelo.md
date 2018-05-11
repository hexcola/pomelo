# Lordofpomelo
Lordofpomelo (lordofpomelo) is a distributed MMO RPG game Demo based on pomelo framework.

## About lordofpomelo
Lordofpomelo covers core content of the mainstream MMORPGs: it has three game scenes, eight different role types, various game tasks, and a variety of items and weapons, as well as a variety of monsters and boss. Players can carry out various kinds of tasks, upgrade their level and interact with other players.

In order to test the performance of pomelo framework, lordofpomelo use a real-time battle mode: most of the player's behaviors, including attack, use skills, the use of items and so on, are excuted in real time. 

Lordofpomelo is developed under pomelo framework standard. By using server clusters instead of a single serve, the capability can be increased by adding more game servers. After multiple rounds of optimization，we can support 800 players in a single scene with a good response time, specific optimization process can be see at (Link).

## Overall Architecture

<img src="http://pomelo.netease.com/resource/documentImage/lordofpomelo/lordofpomelo-all-arch.png" alt="lordofpomelo all architecture" width="600px"></img>

As shown above, lordofpomelo including two types of servers: game-server and web-server. web-server is a http web server, which used as the entry of the game demo. It also includes player register and OAuth authentication logic. After player complete registration and verification, he(or she) will enter the game-server, which supports real game experience. Game-server is the most important part in lordofpomelo, it includes a websocket server cluster as front-end server and a game logic server cluster as backend server, game-server architecture is shown below:

<img src="http://pomelo.netease.com/resource/documentImage/lordofpomelo/game-server.png" alt="lordofpomelo game server" width="600px"></img>

The Clients in the above picture can be any client that support html5 and websocket, it can running in a PC browser, or at any html5 compatible terminal(such as iphone or high performance android device). Players in different platforms can interaction with each others.

The Game-server also can be divided into two different types: Front-end server and Backend Server. Front-end server is a websocket server cluster, it is used to communicate with websoket clients, response the client request and forwarding or filtering message.Frontend servers are also used to broadcasting messages to client.

Backend Servers are mainly used to process the game logic, including various games server types. Such as area server, pathfinding server, chat server and so on.

## lordofpomelo analysis
* [Start process of lordofpomelo] (https://github.com/NetEase/pomelo/wiki/Start-process-for-lordofpomelo)
* [Code structure for lordofpomelo] (https://github.com/NetEase/pomelo/wiki/Lordofpomelo-Code-architecture)
* [Introduction to servers in lordofpomelo] (https://github.com/NetEase/pomelo/wiki/Introduction-to-servers-in-lordofpomelo)