Here are the all runtime configurations of Pomelo.

# 1. masterConfig
This config can only be applied to master server.

### authUser
* Desc: The function to auth user.
* Type: Function.
* Default value: utils.defaultAuthUser from 'pomelo-admin'.

### authServer
* Desc: The function to auth server.
* Type: Function.
* Default value: utils.defaultAuthServerMaster from 'pomelo-admin'.

### Example
``` javascript
app.configure('production|development', 'master', function(){
  app.set('masterConfig', {
    authUser: function () { ... },
    authServer: function () { ... }
  });
});
```

# 2. proxyConfig
The config affects the client of 'pomelo-rpc'. It can be applied to all servers except master server.

### bufferMsg
* Desc: This flag determines whether the pomelo-rpc sends the message directly or not. If true, it will send the message every 'interval (see below)' milliseconds. If false, it will send the directly.
* Type: Boolean.
* Default value: Not set, which will be evaluated as false.

### interval
* Desc: See above. The unit is millisecond. Used when bufferMsg is true.
* Type: Number.
* Default value: 50.

### timeout
* Desc: The seconds that failure action will fire after rpc was called. Unit is second.
* Type: Number.
* Default value: 30.

### Example
``` javascript
app.configure('production|development', function(){
  app.set('proxyConfig', {
    bufferMsg: true,
    interval: 20
  });
});
```

# 3. remoteConfig
The config affects the server of 'pomelo-rpc'. It can be applied to all servers except master server..

### bufferMsg
* Same as proxyConfig.bufferMsg.

### interval
* Same as proxyConfig.interval.

### whitelist
* Desc: Function to provide valid IPs.
* Type: Function.
* Default value: Not set.

### Example
``` javascript
app.configure('production|development', function(){
  app.set('remoteConfig', {
    bufferMsg: true,
    interval: 20
  });
});
```

# 4. connectionConfig
Not implemented.

# 5. connectorConfig
This config is for connector. It can only be applied to frontend servers.

### encode
* Desc: Function to encode data which will be sent over network.
* Type: Function.
* Default value: Provided by connector.

### decode
* Desc: Function to decode data received from network.
* Type: Function.
* Default value: Provided by connector.

### useCrypto
* Desc: If true, pomelo will use RSA to verify the integrity of the message sent by client.
* Type: Boolean.
* Default value: Not set, which will be evaluated as false.

### blacklistFun
* Desc: Function to filter IPv4 address.
* Type: Function.
* Default value: Not set (Do **not** use this feature, use 'iptables' to filter IPs).

If you insist to use this feature, Here is one implementation:
``` javascript
function blacklistFun (cb) {
  var badIpList = ['192.168.1.1', '192.168.1.2'];
  cb(null, badIpList);
}
```

### useDict
* Desc: Enable route compression or not. If true, pomelo will try to read dictionary from $APPBASE/config/dictionary.json. You can override this by providing dictionaryConfig. See below.
* Type: Boolean.
* Default value: Not set, which will be evaluated as false.

### useProtobuf
* Desc: Enable protobuf or not. If true, pomelo will try to read config from $APPBASE/config/clientProtos.json and $APPBASE/config/serverProtos.json. You can override this by providing protobufConfig. See below.
* Type: Boolean.
* Default value: Not set, which will be evaluated as false.

### connector
* Desc: The type of connector.
* Type: Object.
* Default value: sioconnector.

### heartbeat
* Desc: The interval to send heartbeat packet. Unit is **second**.
* Type: Number.
* Default value: Not set.

### disconnectOnTimeout
* Desc: Disconnect the socket if not receive the heartbeat in time.
* Type: Boolean.
* Default value: Not set.

### timeout
* Desc: Seconds to receive heartbeat. Used when disconnectOnTimeout is true. Unit is **second**.
* Type: Number.
* Default value: Not set.

### checkClient
* Desc: Function will be called at handshake.
* Type: Function.
* Default value: Not set.

### handshake
* Desc: Handshake function.
* Type: Function.
* Default value: Not set.

### setNoDelay
* Desc: Disable nagle's algorithm.
* Type: Boolean
* Default value: Not set.

### Example
``` javascript
app.configure('production|development', 'gate|connector', function(){
  app.set('connectorConfig', {
    connector: pomelo.connectors.hybridconnector,
    heartbeat: 45,
    timeout: 55,    // Seconds, heartbeat timer + timeout timer
    disconnectOnTimeout : true,
    useDict: true,
    useProtobuf: true
  });
});
```

# 6. dictionaryConfig
Useful when connectorConfig.useDict is true. It can only be applied to frontend servers.

### dict
* Desc: Absolute path of the dictionary file.
* Type: String.
* Default value: Not set.

### Example
``` javascript
app.configure('production|development', 'connector', function(){
  app.set('connectorConfig', {
    ...
    useDict: true 
  });

  app.set('dictionaryConfig', {
    dict: '/home/foo/config/dictionary.json'
  });
});
```

# 7. protobufConfig
Useful when connectorConfig.useProtobuf is true. It can only be applied to frontend servers.

### serverProtos
* Desc: Absolute path of the server proto file.
* Type: String.
* Default value: Not set.

### clientProtos
* Desc: Absolute path of the client proto file.
* Type: String.
* Default value: Not set.

### Example
``` javascript
app.configure('production|development', 'connector', function(){
  app.set('connectorConfig', {
    ...
    useProtobuf: true
  });

  app.set('protobufConfig', {
    serverProtos: '/home/foo/config/serverProtos.json',
    clientProtos: '/home/foo/config/clientProtos.json'
  });
});
```

# 8. sessionConfig
Config for frontend sessionService. It can only be applied to frontend servers.

### singleSession
* Desc: [According to the doc](https://github.com/NetEase/pomelo/wiki/Backend-server), if set it to true, then it allows only one logged session for an user, when a new session is logging, the logged session will be kicked. **But from the codes, if set it to true, the logged session is kept, and the new session is denied.**
* Type: Boolean.
* Default value: Not set which will be evaluated as false.

### Example
``` javascript
app.configure('production|development', 'connector', function(){
  app.set('sessionConfig', {
    singleSession: true
  });
});
```

# 9. schedulerConfig / pushSchedulerConfig
Config for push scheduler. It can only be applied to frontend servers.

### flushInterval
* Desc: Push message to clients every 'flushInterval' milliseconds.
* Type: Number.
* Default value: 20.

### Example
``` javascript
app.configure('production|development', 'connector', function(){
  // app.set('pushSchedulerConfig', {
  app.set('schedulerConfig', { 
    flushInterval: 10
  });
});
```

# 10. backendSessionConfig
Not implemented.

# 11. channelConfig
Config for channelService. It can be applied to all servers except master server.

### prefix
* Desc: A string will be used to create object key.
* Type: String.
* Default value: Not set which will be evaluated as ''.

### store
* Desc: An object to store channelService. Seems like redis or something else.
* Type: Object.
* Default value: Not set.

### broadcastFilter
* Desc: A function which will be called when push messages to clients if set.
* Type: Function.
* Default value: Not set.

# 12. serverConfig
Config for servers. It can be applied to all servers except master server.

### reloadHandlers
* Desc: This flag controls whether pomelo will reload handlers when they are modified.
* Type: Boolean.
* Default value: Not set which will be evaluated as false.

### Example
``` javascript
app.configure('production|development', 'connector', function(){
  app.set('serverConfig', { 
    reloadHandlers: true
  });
});
```

# 13. monitorConfig
Config for monitor module. It **should** be applied to all servers including master server if you want to use this config.

### closeWatcher
* Desc: Used to enable/disable the watchdog function.
* Type: Boolean.
* Default value: Not set which will be evaluated as false.

### Example
``` javascript
app.configure('production|development', function(){
  app.set('monitorConfig', { 
    closeWatcher: true
  });
});
```