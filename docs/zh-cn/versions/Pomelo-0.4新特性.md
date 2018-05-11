#Pomelo0.4新特性

在0.4版本中，主要对Pomelo内部结构进行了调整，提供更友好的自定义通讯协议支持，以及自定义的数据发送包调度策略，为各种应用场景提供更灵活的支持。

##自定义协议

###服务器端实现

根据网友的建议，Pomelo框架对协议模块做了更细致的划分，以支持更灵活的定制特性。主要结构如下图所示：

<center>
![pomelo-transport](http://pomelo.netease.com/resource/documentImage/pomelo-transport.png)
</center>

在上层模块和底层的transport模块之间加入protocol层对协议数据包进行编解码。Protocol层主要针对上层通讯数据进行编解码，并与具体的connector实现相关联。对于`hybridconnector`，编解码的层次对应于Message层的协议数据。对于`sioconnector`，编解码的层次则对应于socket.io协议之上的数据。

Protocol层的编解码算法可以通过向`connector`组件传递encode/decode初始化参数来定制。具体例子如下：

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

每当connector上有数据包需要编解码即会触发对应的encode/decode函数。

* `encode(reqId, route, msg)` - 负责将`reqId`, `route`和`msg`进行编码。可以通过`reqId`对判断协议包的类型，如果`reqId > 0`，表示是request消息，如果`reqId === 0`，表示是push消息。返回值即编码结果，将直接投递给底层传输模块发送出去。

* `decode(msg)` - 负责将底层获取到的数据包解码成上层的消息。`msg`为底层传输模块获取到的数据，与客户端相同层次所传输的数据一致。返回结果为解码后的结果，是上层框架所能处理的消息，格式为：{id: id, route: route, body: body}。其中id为消息的id，request类型的消息id为正整数，notify消息可以没有id字段。route为String类型，消息的route字段；body为Object类型，即消息体。

可以通过 `pomelo.connectors.hybridconnector.encode/decode` 和 `pomelo.connectors.sioconnector.encode/decode` 获取对应connector的默认编解码方法。

###JS客户端实现

与服务器端类似，Pomelo客户端也提供了相应的定制参数，具体例子如下：

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

##数据包发送的调度

从上面的图可以看到，消息经过protocol层编码后，还需要经过一个scheduler层才能到达最下面的transport层。

scheduler层负责决定消息发送的调度策略，比如：消息的发送优先级，广播时的分批发送等。

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

schedule(route, msg, recvs, opts, cb)负责实现具体的调度逻辑。在schedule函数内部可以调用`sessionService.sendMessage`将消息发送出去。

* route - 消息的route字符串，response消息route参数为空。
* msg - 编码后的消息内容。
* recvs - 接收者的session id列表。
* opts - 附加参数。opts.isBroadcast表示当前消息是否是广播，opts.isResponse表示当前消息是否是response，opts.isPush表示当前消息是push消息。

Pomelo框架默认提供两类scheduler。

* `pomelo.schedulers.direct` - 默认采用的scheduler，直接将上层下来的消息发送出去。
* `pomelo.schedulers.buffer` - 对上层的消息进行缓存，并定期批量发送出去。可以通过`flushInterval`初始化参数设置刷新间隔，单位为毫秒，默认值为20毫秒。


##支持同一账号多处登录

0.4版本之前同一个账号在同一个前端服务器中只能存在一个session。0.4之后支持同一个账号多地登录，以支持在web应用中多个页面登录应用的情景。

该特性会影响到下列接口：

* `localSessionService.getByUid` - 语义改为获取指定uid下所有的session信息，返回的session改为session数组。
* `localSessionService.kickByUid` - 语义改为踢对应uid所有的session下线。

以及`sessionServie`中的对应私有方法：

* `sessionService.getByUid` - 语义改为获取指定uid下所有的session信息，返回的session改为session数组。
* `sessionService.kickByUid` - 语义改为踢对应uid所有的session下线。

##HybridConnector添加心跳超时断开连接选项

0.3版本中，`hybridconnector`服务器端检测到心跳超时不会主动断开连接。在0.4.2版本中，添加`disconnectOnTimeout`选项，当服务器检测到心跳超时后会主动断开该客户端连接。使用的例子如下：

```javascript
  app.set('connectorConfig', {
    connector : pomelo.connectors.hybridconnector,
    heartbeat : 3,
    disconnectOnTimeout: true
  });	
```