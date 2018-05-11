Pomelo-daemon provides a daemon service for distributed deployment of pomelo application and collecting RPC logs of servers.

Installation
=================

    $ npm install -g pomelo-daemon

Usage
========

#### Starting Pomelo Server Cluster

* Deploy codes on all the servers of the server cluster;
* Configure servers.json to right host; 

* Add daemon.json configuration file into game-server/config, A sample of daemon.json is shown as follows:

```json
{
  "id": "dh37fgj492je",
  "key": "agarxhqb98rpajloaxn34ga8xrunpagkjwlaw3ruxnpaagl29w4rxn",
  "algorithm": "sha256",
  "user": "pomelo"
}
```

**Note**: pomelo-daemon uses [hawk](https://github.com/hueniverse/hawk) to provide authentication between servers. 
* Change current directory to game-server; 
* Execute the following command in the master server:
```
pomelo-daemon
```
* In other servers, execute: 
```  
pomelo-daemon --mode=server
```
**Note**: You can use nohup to deploy daemon as follows:
```
nohup pomelo-daemon - mode = server
```
* Then in the master server run the following command under pomelo-daemon client:
```  
start all
```
* After all above, the pomelo server cluster is launched. 

#### Collecting RPC Logs  
pomelo-daemon provides a pomelo rpc logs collector to synchronize the the rpc logs to mongodb, then it can be analyzed by [pomelo-admin-web] (https://github.com/NetEase/pomelo-admin-web).

* Add mongo.json configuration file into game-server/config, A sample of mongo.json is shown as follows:

```json
{
  "host": "localhost",
  "port": 27017,
  "username": "pomelo",
  "password": "pomelo",
  "database": "test",
  "collection": "cpomelo"
}
```
* Enable pomelo-daemon rpc logger collecting by setting `--pattern` parameter to rpc-log: 
```  
    pomelo-daemon --mode=server --log - pattern = rpc-log
```
**Note**: RPC logs collected here is only used for debugging in development environment, it is not recommended to enable RPC logs in production environemnt. 

