# Command Line Document   
Pomelo includes various shell-like commands that support application operations like creating a new project, starting the server application, debugging a running application, stopping a running application, killing a running application, etc. 

## Installation

    npm install pomelo -g

#### `pomelo init <dirname>`

Create a new project under the path you give. If the path is not provided, the default path is current path, the default project name is current directory name; relative path is also supported.

#### `pomelo start [-e <environment>] [-debug] [--trace] [--profile] [--daemon]`

Launches the application and servers. Can be run in `game-server/` or `web-server/`.

`--daemon`: runs the application in the background. Log files by default are stored in `game-server/logs`. Application running in daemon need to install [forever](https://www.npmjs.org/package/forever).

`–debug`: If need to debug in a certain server, just add the parameter '--debug=[port]'.For example, if you need to debug at 8080, --debug=8080. The supported parameters can refer to the command line parameters of node.js and v8. The command must be used in 'game-server' directory of the application.

`--trace`

`--profile`

`-e <environment>`: environment is normally 'development' (default) or 'production', although you can set up custom environments by editing `/game-server/config/servers.json` and `/game-server/app.js`. For example：

    {
      "connector":[{
        "id": "connector-server-1",
        "host":"127.0.0.1",
        "port":4050,
        "wsPort":3050,
        "args":"--debug=[port] --trace"}
      ]
    }

#### `pomelo add host=[host] port=[port] id=[id] serverType=[serverType]`

Add a server at runtime. The command must be used from the master server.    

#### `pomelo list [-P <port>]`

List all servers' information during runtime. The command must be used in root directory of the application. If your master server is running on a port other than the default, use the `-P` option to specify listing all servers running off the master server on the provided port.

#### `pomelo stop [-P <port>]`

Close all servers and stop the application gracefully. Unlike the kill command, it will first close the connection between clients and the server, and then terminal all servers. The command must be used in root directory of the application. Like the `list` command, you can specify a different master server port if necessary.

#### `pomelo kill [--force] [-P <port>]`

Close the application forcefully.If happens that there left application processes after killing when you develop in local, you can use the command `pomelo kill --force` to kill all application processes. It may lead to bad influences like data loss, so we do not recommend using this command in production mode. The command must be used in root directory of the application.

Like the `list` command, you can specify a different master server port if necessary.

#### `pomelo --version`

List the current version of pomelo. The command can be used in the global.

#### `pomelo --help`

List all the commands currently supported by pomelo. The command can be used in the global.