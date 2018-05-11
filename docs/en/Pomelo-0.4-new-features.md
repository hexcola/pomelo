# New features in Pomelo 0.4

Pomelo 0.4 makes structural changes to support more friendly custom communication protocol, and provides custom data transmission strategies for different situations.

## Custom protocol

### Implementation of server side

Pomelo 0.4 makes a more detailed protocol division to support more flexible custom features. The main structure is as follow chart: 

<center>
![pomelo-transport](http://pomelo.netease.com/resource/documentImage/pomelo-transport.png)
</center>

We add the protocol layer to encode/decode data packages between the upper layer and the under layer. The protocol layer is mainly in charge of encoding and decoding transmission data for the upper layer, and is associated with specific connector. Hybridconnector is responsible for dealing with data from the message layer, and sioconnector is responsible for dealing with data from the socket.io layer.

Encoding and decoding algorithm in the protocol layer can be custom made by passing it to connector, code example is as follow:

```javascript
var encode = function(reqId, route, msg) {
  // do some customized encode with reqId, route and msg
  return result;	// return encode result
};

var decode = function(msg) {
  // do some customized decode with msg
  return result;	// return decode result
};

app.configure('production|development', 'connector', function(){
  app.set('connectorConfig', {
    connector : pomelo.connectors.hybridconnector,
    encode: encode,
    decode: decode
  });
});
```

When data comes to a connector, corresponding encode and decode methods will be invoked.

* `encode(reqId, route, msg)` - means encoding `reqId`, `route` and `msg`. We can know package type by checking `reqId`, if `reqId > 0` it represents request message, if `reqId === 0` it represents push message. And this method's return value is the result of encoding message, which will be passed to the transmission layer to send out.

* `decode(msg)` - is responsible for decoding messages from the under layer. `msg` is the data that the under layer receives, which is in accordance with the data in the same layer of client side. This method's return value is the result of decoding message, which can be handle by the upper layer, and it format is :　{id: id, route: route, body: body}． In this kind of message, id is the identity of message, request message's id must be positive integer, notify message does not have this field, the type of route field is string which means message router, the type of body field is object which means message body.

We can get default encode and decode method by `pomelo.connectors.hybridconnector.encode/decode` and  `pomelo.connectors.sioconnector.encode/decode`.

### Implementation of JS client

Client side is similar to server side, which also provides corresponding custom parameters, code example is as follow:

```javascript
var encode = function(reqId, route, msg) {
  // do some customized encode with reqId, route and msg
  return result;	// return encode result
};

var decode = function(msg) {
  // do some customized decode with msg
  return result;	// return decode result
};

pomelo.init({
  host: host, 
  port: port, 
  log: true, 
  encode: encode, 
  decode: decode}, function() {
});
```

## Schedule of data transmission

As we can see from the chart above, there is the scheduler layer between the protocol layer and the transport layer.

The scheduler layer is deciding the data sending strategy, for example, the sending priority and batch operations in broadcasting messages.

```javascript
var SchedulerService = function() {
};

SchedulerService.prototype.schedule = function(route, msg, recvs, opts, cb) {
  // do the schedule
};

app.configure('production|development', 'connector', function(){
  app.set('schedulerConfig', {
    scheduler: new SchedulerService()
  });
});
```

schedule(route, msg, recvs, opts, cb) is responsible for implementation of schedule strategy. You can use `sessionService.sendMessage` to send messages in schedule method.

* route - message router, and in response message this field is null.
* msg - messages after encoding.
* recvs - receivers's session.id list.
* opts - additional parameters. opts.isBroadcast means the message is need to do broadcast operation，opts.isResponse means the message is need to do response operation，opts.isPush means the message is need to do push operation.

Pomelo provides two kinds of scheduler.

* `pomelo.schedulers.direct` - Default scheduler，which will send messages directly.
* `pomelo.schedulers.buffer` - Optional scheduler, which will cache messages and send messages in batch. You can adjust messages sending interval by modifying `flushInterval`, and its default value is 20ms.

## HybridConnector heartbeat timeout

In Pomelo 0.3 `hybridconnector` will not close the connection after heartbeat timeout. In Pomelo 0.4.2, we add the option `disconnectOnTimeout`, when the server side detecting heartbeat timeout it will close the client connection, code example is as follow:

```javascript
  app.set('connectorConfig', {
    connector : pomelo.connectors.hybridconnector,
    heartbeat : 3,
    disconnectOnTimeout: true
  });	
```