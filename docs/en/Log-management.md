Log Management
====================

Log in pomelo is maintained by [pomelo-logger] (https://github.com/NetEase/pomelo-logger), which is a simple wrapper of [log4js] (https://github.com/nomiddlename/log4js-node) and provides some very useful additional features.

Logs can be divided into categories to maintain, and categories can be configured in log4js.json file, Here is an example as follows: 

```json
{
  "appenders": [
    {
      "type": "console"
    },
    {
      "type": "file",
      "filename": "./logs/con-log-${opts: serverId}.log",
      "pattern": "connector",
      "maxLogSize": 1048576,
      "layout": {
        "type": "basic"
      },
      "backups": 5,
      "category": "con-log"
    },
    {
      "type": "file",
      "filename": "./logs/rpc-log-${opts: serverId}.log",
      "maxLogSize": 1048576,
      "layout": {
        "type": "basic"
      },
      "backups": 5,
      "category": "rpc-log"
    },
    {
      "type": "file",
      "filename": "./logs/forward-log-${opts: serverId}.log",
      "maxLogSize": 1048576,
      "layout": {
        "type": "basic"
      },
      "backups": 5,
      "category": "forward-log"
    },
    {
      "type": "file",
      "filename": "./logs/rpc-debug- $ {opts: serverId}. log",
      "maxLogSize": 1048576,
      "layout": {
        "type": "basic"
      },
      "backups": 5,
      "category": "rpc-debug"
    },
    {
      "type": "file",
      "filename": ". / logs / crash.log",
      "maxLogSize": 1048576,
      "layout": {
        "type": "basic"
      },
      "backups": 5,
      "category": "crash-log"
    },
    {
      "type": "file",
      "filename": ". / logs / admin.log",
      "maxLogSize": 1048576,
      "layout": {
        "type": "basic"
      }
      "backups": 5,
      "category": "admin-log"
    },
    {
      "type": "file",
      "filename": ". / logs / pomelo.log",
      "maxLogSize": 1048576,
      "layout": {
        "type": "basic"
      }
      "backups": 5,
      "category": "pomelo"
    }
  ],

  "levels": {
  "rpc-log": "ERROR",
  "forward-log": "ERROR"
  },
  "replaceConsole": true,
  "lineDebug": false
}

```

As can be seen from the configuration file, each configuration item (except console) is configured with a category. A logger can be obtained by calling **getLogger** method of pomelo-logger with a specific category as its first argument, and then the logger will output the generated logs to file or other device based on the configuration in the configuration file of that category.

You can also add your own log category, and configure how to output the logs of that category, and then call **getLogger** to get the logger for that category, and the logger can be used in your project.

**Note**: It is not recommended to use logger that belongs to no category, if so, the log will be output based on global configuration.

### Log Categories
There are several categories configured in the configuration file, each category indicates a logger for specific purpose:

* pomelo: logger for pomelo framework;
* admin-log: logger for server management framework;
* crash-log: logger for server exception; 
* rpc-debug: logger for RPC invocation, it is required to enable [rpc-debug feature] (https://github.com/NetEase/pomelo/wiki/Pomelo%27s-new-features-in-version-0.6) first;
* forward-log: logger for frontend servers forwarding requests from clients to backend servers; 
* rpc-log: logger for RPC filter; 
* con-log: logger for handler filter. 

### Log Levels
Log levels indicate the severities of logs, it can be used to determine log output, Here is an example:

```javascript
"levels": {
  "rpc-log": "ERROR",
  "forward-log": "ERROR"
}
```

The following presents all the log levels, their severity is becoming higher and higher from left to right:

```
TRACE, DEBUG, INFO, WARN, ERROR, FATAL
```
The lower is the level configured for a category, the more log will be output, and vice versa. For example:

```javascript
var rpc_logger = require('pomelo-logger').getLogger ('rpc-log', __filename);
rpc_logger.info ("msg");
```

Here rpc_logger uses **info** method, meaning the level is INFO, however the levels of that category is configured to ERROR, so the log would not be output. 

But if you change the levels configuration for rpc_logger, make it be below INFO level such as DEBUG, and then the log will be output.

### Configuration Items 
* type: It specifies the type of appender, it can be console, dataFile, file , etc., more detail at [log4js] (https://github.com/nomiddlename/log4js-node/wiki/Appenders);
* filename: It specifies the output file path;
* pattern: It specifies the pattern of log output;
* maxLogSize: It specifies the maximum size of log output;
* layout: It specifies the layout style for outputting log;
* backups: It specifies the maximum number of output files;
* category: It specifies the corresponding category to the appender. Non-configured on it indicates that the appender is a global appender;
* replaceConsole: It specifies whether to replace the default console;
* lineDebug: It specifies whether to enable debug to show the number oflog lines. 