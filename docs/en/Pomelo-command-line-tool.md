Pomelo provides command line tool that can help developers build applications  more conveniently and more efficiently. The tool can be used for various development operations, including creating project, starting application, debugging application, stopping application, etc..

Tool Installation
===========

Pomelo command line tool will be automatically installed when installing pomelo framework using following command:

    $ npm install pomelo -g

Commands
==========

Currently pomelo supports the following commands:

* init: Create a new project that contains basic files with which can build pomelo-based applications;
* start: Launch the application and the server;
* list: List information of all servers that the application is running on, including serverId, serverType, pid, heap usage and uptime.
* stop: Stop all or specified running server of the application.
* kill: Kill the application immediately in a rude way.
* add: Dynamicly add an application server at runtime.
* --version: Show version of pomelo framework.
* --help: Show information of all commands and instructions supported by pomelo framework.


Command Usage 
========================
* init

It is used to create a new project at the path given by user, and it supports relative and absolute paths. The default path is the current work directory and the default project name is the name of the current work directiory:

    pomelo init [dirname]

**Note**: When creating a new project, you need to choose which connector to use when communicating with the client, 1 for Websocket(native socket), 2 for socket.io. 

* start

The command is to launch pomelo application:
    
    pomelo start [-e, --env <env>] [-d, --directory <code directory>]
                 [-D, --daemon]

Where, option `-e, --env` is used to specify environment and it would be `development` by default; `-D, --daemon` is to enable daemon mode, if it is enabled, then the application would run in background; `-d, --directory` is used to specify where the source code places and it would be current work directory by default.

**Note**: Prior to version 0.8, `pomelo start` does not support `-e, --env` option, and it use the way below to specify the env:

    pomelo start production

`forever` is not requireed in version 0.8 if starting pomelo appliction in daemon mode.

Users can add parameters for different type of servers by modifying server configuration file <project_dir>/game-server/config/servers.json. The parameter for v8/node is also supported while launching servers. For example, when debugging a server is required, we can configure the server starting parameters shown as follows:

```json
{
  "connector": [
    {
      "id": "connector-server-1", "host": "127.0.0.1", "port": 4050,
      "clientPort": 3050, "args": "--debug=[port]"
    }
  ]
}
```

* list

This command can list information of all running servers. When doing this, the tool is as a client for server administration framework, so username/password is required to register to master server. For the default generated project by `pomelo init`, there is a default user named `admin`, whose password is also `admin`. So that if username/password is not specified, the default username/password will be both `admin`. The following `stop` command and `kill` command require username/password too, and the default is the same as `list` here. Developers can modify file config/adminUser.json to configure the admin users, and the generated project has a sample for that.

It is also required to specify the host:port of master server, so that this command can be executed everywhere, and so does `pomelo stop/kill/add`.  The default is 127.0.0.1:3005.

    pomelo list [-u, --username <username>] [-p, --password <password>]
                [-h, --host <master-host>] [-P, --port <master-port>]

* stop

This command can stop the application gracefully. Being different from `pomelo kill` command, it will first close the client connections between clients and the server, and then terminate all servers one by one. You can also stop only a server not all servers by specifying a serverId.Being similar as `list` command, this command requires username/password and master's host:port  specified too. 

    pomelo stop [-u, --username <username>] [-p, --password <password>]
                [-h, --host <master-host>] [-P, --port <master-port>]
                [<serverIds>...]
Example:
    
    pomelo stop                     # stop all the servers
    pomelo stop server-1 server-2   # just stop server-1 server-2

* kill

This command can teminate the application forcefully. If the application is running locally in development environment, you can use `pomelo kill --force` to kill all the node processes, which is quite violent and it may result in data loss and other bad effects. You can use this command when developing and debugging, it is not recommended to use this command in production environment. The command also requires authentication same as `list` and `stop`.

    pomelo kill [-u, --username <username>] [-p, --password <password>]
                [-h, --host <master-host>] [-P, --port <master-port>]
                [-f, --force]

* add

This command can add server dynamicly at runtime. Being similar to `pomelo list`, it also requires authentication and master server host:port specified.

    pomelo add [-u, --username <username>] [-p, --password <password>]
                [-h, --host <master-host>] [-P, --port <master-port>]
                [<server-args>...]
Example:
    
    pomelo add host=127.0.0.1 port=8000 clientPort=90000 frontend=true serverType=connector id=added-connector-server-1

* masterha
 
If master HA is enabled, this command can start the slaves of the master server, it needs the configuration file masterha.json placed into game-server/config. `-d, --directory` for code directory and by default it would be current work directory:
    
    pomelo masterha [-d, --directory <code directory>]

Example:

    pomelo masterha 
    pomelo masterha -d /home/my/project/game-server

* other

To view the current version of pomelo:
  
    pomelo --version

Help info:
    
    pomelo --help
    pomelo add --help
    pomelo kill --help
