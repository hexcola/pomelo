Pomelo-cli is a monitoring and management client which provides an interactive command line tool to help developers maintain the applications based on [pomelo] (https://github.com/NetEase/pomelo) framework.

Installation
=============
```
npm install -g pomelo-cli
```

Connecting and Registering to Master
====================================
```
pomelo-cli -h <master host> -P <master port> -u <username> -p <password>
```

If you use `pomelo-cli` without parameters, the default parameters for pomelo-cli is as follows: 
```
pomelo-cli -h 127.0.0.1 -P 3005 -u monitor -p monitor
```
The access control is configured in config/adminUser.json, please refer to [admin access control] (https://github.com/NetEase/pomelo-admin#user-level-control) for more detail.

The commands such as kill, stop, add, enable, disable is in restricted to non-admin user.

Commands
==========

#### use
Switch context for the cli environment, the new context can be serverId or all. After switching context, the following command will be applied to the new context util switching context to others, and all means global context, as below:

```
 > use [ <serverId> | all ]

example:
 > use area-server-1
 > use all
```

#### quit

exit pomelo-cli.

#### kill
Close all servers.

**Note**: Be careful to use this command.

#### exec
Execute the specified script file in servers and then return the result.
```
> exec <filepath>
```
Here, filepath can be a relative path to pomelo-cli executing path or an absolute path. For example:
```
example:
> exec xxx.js
> exec /home/user/xxx.js
```
<a name="spec" />
The script file is executed by [vm] (http://nodejs.org/api/vm.html) module provided by node as a sandbox, and its executing context is:
```javascript
var context = {
  app: this.app, // pomelo application instance
  require: require, // global function of node  
  os: require("os"), // os module of node
  fs: require("fs"), // fs module of node
  process: process, // process module of node
  util: util // util module of node
};
```
Result for the script executing is passed out by a global variable **result**, that means you should assign the executing result to **result** variable. Here is a sample script: 
```javascript
// getCPUs.js
var cpus = os.cpus();

// declare result without `var` to make it be global
result = util.inspect(cpus, true, null); 
```

#### get
Equals to app.get(key).
```
> get <key>
```

#### set
Equals to app.set(key, value)
```
> set <key> <value>
```

**Note**: Value here must be JSONfied. 

#### add
Add a server to pomelo cluster dynamically at runtime. Arguments for this command are shown as follows, they should be a set of key-value pair same as the configuration information in servers.json:

```
example:
> add host=127.0.0.1 port=3451 serverType=chat id=chat-server-2
> add host=127.0.0.1 port=3152 serverType=connector id=connector-server-3 clientPort=3012 frontend=true
```

**Note**: The complete arguments for a server is required when adding a new server, otherwise, the added server will run in a ill state.
#### stop
Stop server specified by its argument.
```
> stop <serverId>

example: 
> stop area-server-1
```

#### show
Show information such as servers, connections, etc.
```
> show [servers|connections|logins|modules|status|proxy|handler|components|settings]

example:
> show servers
> show connections
> show proxy
> show handler
> show logins
```

**Note**: This command should be executed under a certain server context not global context except `show servers`, which can be executed under any contexts. 

#### enable
Enable an admin module or enable an app settings. 
```
> enable [ module <moduleId> | app <settingKey> ]

example:
> enable module systemInfo
> enable app systemMonitor
```

#### disable
Oppisite to enable.

```
> disable [ module <moduleId>|app <settingKey> ]

example:
> disable module systemInfo
> disable app systemMonitor
```

#### dump
Make a dump of v8 heap and cpu for later inspection. 
```
> dump [ cpu | memory ] <filepath> <times> [--force]

example:
> dump cpu /home/xxx/ test 5
> dump memory /home/xxx/ test
```
**Note**: times argument is used in cpu dump indicates how long does cpu dump spends in seconds, filepath indicates where to place the dump file. You can use the --force to overwrite the existed file.
```
example: 
> dump cpu /home/xxx/ test 5 --force
> dump memory /home/xxx/ test --force
```
After dumping for cpu or v8 heap, you can inspect the dump file via following steps:
* Open [google chrome] (https://www.google.com/intl/en/chrome/browser/) browser and press F12 to open the developer toolbar; 
* Go to **Profiles** tab, right-click in the tab pane and select **Load profile...**;
* Select the dump file and click **Open**.

For more detail about dump, you can visit [ndump] (https://github.com/piaohai/ndump).
