This document will guide you through the process of installing Pomelo, show you how to create your first application & explain some of the core concepts behind the framework itself. 

Pomelo comes with a command line utility which provides a range of commands to simplify the development process. All these commands are thoroughly tested on linux, mac and windows os.

## Installing Pomelo

The following process is somewhat misleading, refer to this page instead. [Installation](https://github.com/NetEase/pomelo/wiki/Installation)

> This guide assumes you have already installed `node` & `npm`.

> **Linux / OSX / Windows**:
> ```bash
> $ npm install pomelo -g
> ```

> **Windows**:

>Make sure you have installed `node.js source code compiling tools`.

>node.js uses [gyp](http://code.google.com/p/gyp/) to generate the Visual Studio Solution file, which is compiled to a binary file by the VC++ compiler.

>Minimum requirements for installing Pomelo on windows:
> * [Python](http://python.org/) ( 2.5 < version < 3.0 ).
> * VC++ compiler, included in [Visual Studio 2010](http://msdn.microsoft.com/en-us/vstudio/hh388567) or VC++ 2010 Express.

> Then install the Pomelo command line utility.
> ```bash
> > npm install pomelo-cli -g
> ```


## Usage

### Creating a new project

Pomelo provides an interactive installer which will guide you through the process & install all the files you will need for your first project.

To create your first project:
```bash
$ cd /var/www/
$ pomelo init helloWorld
$ cd helloWorld/
$ sh npm-install.sh 
```
 

A new project is made up of the following directories:

![directory structure](http://pomelo.netease.com/resource/documentImage/helloWorldFolder.png)

Your new application is split in to 2 main directories called `game-server` and `web-server`.

| Directory | More Info |
|:----------------------------------------- |:----------- |
| **/game-server** | Contains all server side code related to the game dynamics & components. |
| **/web-server** | Contains a web server based on expressjs for serving client-side code & web content.  |
| /game-server/config | Contains configuration files to set up & tweak your server-side environment. |
| /logs | Contains all the application logs which prove essential in debugging your application.  |
| /shared | Can be used to share javascript files between the client & server (if applicable).  |


### Start project

Both game-server and web-server should be started, and game-server can be started by  the following command 

`pomelo start [development | production] [--daemon]` 

while web-server is started like this:
 `cd web-server && node app`

Running in different environment, the project needs different start arguments. If the running environment is development, the argument should be development, otherwise it should be production. By default the project runs in foreground model, and the argument '--daemon' can let project run in background model. 

If you use “–daemon” argument, the module 'forever' should be installed by command `npm install forever -g`

You can also start the game-server with this command:

`node app [env=development | env=production]`

After project started, you can visit website of 'http://localhost:3001' or 'http://127.0.0.1:3001' by browser with
websocket support.

### Query server status

You can query server status with the command `pomelo list`, and the example of result is shown as follows:

![test](http://pomelo.netease.com/resource/documentImage/pomeloList.png)

The followings are detailed definition of each fields:

* serverId: the id of server which is the same as configuration
* serverType: the type of server which is the same as configuration
* pid: the pid of process corresponding to server
* headUsed: the used heap size of server(Unit: Megabytes)
* uptime: the duration since this server started(Unit: Minute)

### Shutdown project

Both commands `pomelo stop` and `pomelo kill` can shutdown the project. The command `pomelo stop` is recommended for some
reasons as follows:

* front-end servers disconnect to keep players from coming in 
* Guarantee game logic with closing all of the servers by certain order
* Ensure data integrity with writing plays' information to the database in time 
 
command `pomelo stop id` will dynamically shutdown the server corresponding to the given id.

Please avoid the way with command `pomelo kill` in production environment. Use it only to kill if there is  no other remedy. 

### add server
`pomelo add host=[host] port=[port] id=[id] serverType=[serverType]` can help you add server with the given arguments dynamically. Now this command is only for backend server, and required in the project root directory.

## AdminConsole

AdminConsole is a powerful tool which can monitor the project and get valuable information. You can do the following things with adminConsole.

* Monitor server status, logs, online users and scene data etc.
* Get related information with scripts flexibly.
* Trace and analyze memory stack, CPU consumption 

sysstat tool is required for AdminConsole on linux, so you should install sysstat tool by `apt-get install sysstat` .

### Install adminConsole

>git clone https://github.com/NetEase/pomelo-admin-web.git

>cd pomelo-admin-web

>npm install -d

>node app

If you have not installed sysstat on linux, execute the following command:

>apt-get install sysstat

Open the console by visiting website of 'http://localhost:7001' with browser which supports websocket. If there are any port conflicts, please fix configuration in 'config/admin.json'.The system closes the admin console module in default, so you need to open it in game-server/app.js and add the code app.enable('systemMonitor'). You can refer to the source code of lordofpomelo in detail.
## Project in production environment

If multi-server environment, not only all the servers should support "ssh agent forward", but also the project directory structure of each
server should be exactly the same.

Here are some references about 'ssh agent forward':

* [Getting Started with SSH](http://kimmo.suominen.com/docs/ssh/)
* [What is an SSH Agent](http://en.wikipedia.org/wiki/Ssh-agent)
* [How SSH Agent Forwarding Works](http://unixwiz.net/techtips/ssh-agent-forwarding.html)
* [What are deploy keys?](https://help.github.com/articles/managing-deploy-keys)