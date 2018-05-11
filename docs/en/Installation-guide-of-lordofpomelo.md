Lordofpomelo is a massively multiplayer online role-playing game based on pomelo framework and composed of 
character, equipment, skill, upgrade, task system etc.

It is completed in less than two months and has about 6000 lines codes in server and client side respectively.
Base on pomelo framework on server side and colorbox framework on client side, lordofpomelo supports about 1000 
players concurrent access in one scene and shorten the response time to 200 millisecond.

## Runing environment

* [nodejs](http://nodejs.org/)
* Linux or Mac os
* mysql

## Installation

### Download source

`git clone https://github.com/NetEase/lordofpomelo.git`

### Install dependencies

`cd lordofpomelo && sh npm-install.sh`

### Install components
`npm install -g component`  
`cd lordofpomelo/web-server && sh bin/component.sh`  

## Create database

Create database with file './game-server/config/schema/Pomelo.sql'.

* Install mysql database
* Login database: `mysql -uUsername -pPassword`
* Create database: `mysql> create database Pomelo`
* Choose database: `mysql> use Pomelo`
* Import sql file: `mysql> source ./game-server/config/schema/Pomelo.sql`

### Modify database configuration

The database configuration file is ‘/lordofpomelo/shared/config/mysql.json', and you can modify corresponding parameters according to your own configuration.
```json

    {
      "development": 
      {
         "host": "127.0.0.1",
         "port": "3306",
         "database": "Pomelo",
         "user": "root",
         "password": "123456"
      },
      "production":
      {
         "host" : "127.0.0.1",
         "port" : "3306",
         "database" : "Pomelo",
         "user" : "root",
         "password" : "123456"
      }
    }
```

### Run game

Both game-server and web-server should be started, and game-server can be started by  the following command 

`pomelo start [development | production] [--daemon]` 

while web-server is started like this:
 `cd web-server && node app`

At last, please visit website of 'http://localhost:3001' or 'http://127.0.0.1:3001' by browser with
websocket support. By the way, chrome is recommended.

## Solution for some problems

* port conflicts

Modify configuration in './game-server/config/server.json':

```json
{
  "development": {
    "connector": [
	{"id": "connector-server-1", "host": "127.0.0.1", "port": 3150, "clientPort": 3010, "frontend": true},
	{"id": "connector-server-2", "host": "127.0.0.1", "port": 3151, "clientPort":3011, "frontend": true}
    ],
  "area": [
	{"id": "area-server-1", "host": "127.0.0.1", "port": 3250, "area": 1},
	{"id": "area-server-2", "host": "127.0.0.1", "port": 3251, "area": 2},
	{"id": "area-server-3", "host": "127.0.0.1", "port": 3252, "area": 3}
  ],
  "chat": [{"id":"chat-server-1","host":"127.0.0.1","port":3450}],
  "path": [{"id": "path-server-1", "host": "127.0.0.1", "port": 3550}],
  "auth": [{"id": "auth-server-1", "host": "127.0.0.1", "port": 3650}],
  "gate": [{"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true}]
 },
  "production": { }
}
```

This file regulates the servers configuration information, such as serverType, serverId, host, port etc. The configuration information in production is similar with development, and the frontend is the flag for frontend server. 

Also you can modify configuration of '/web-server/public/js/config/config.js':

```json
IMAGE_URL: "http://pomelo.netease.com/art/",
GATE_HOST: window.location.hostname,
GATE_PORT: 3014
```
This is the most common configuration for web server. You can get static resources like images by IMAGE_URL.
GATE_HOST is the access of websocket request which gets the connectorId from gate server and connects to connector
server. GATE_PORT is the port of gate server.

## Details for configurations

* ./game-server/config/master.json

This file is about master server configuration, including serverId, host etc both in development and production.
The master server is responsible for starting and closing other servers, and also in charge of monitoring state information of them.

* ./game-server/config/servers.json

The configuration information for area, connector and status server in all environments. The connector server has websocket port parameter because it needs to communicate with clients through websocket.

* ./shared/config/mysql.json

The database configuration information. You should modify the configuration parameters after installing lordofpomelo 
in development environment. 

* ./web-server/public/js/config/config.js

This is the most common configuration composed of IMAGE_URL, GATE_HOST and GATE_PORT for web server.

## Supplementary for login

Not only can you create a new account, but also use account of github, google, facebook, twitter and sina weibo after authorized
to access to lordofpomelo. Of course, you should get the authority and modify the configuration parameters locating in './web-server/config/oauth.json'.



