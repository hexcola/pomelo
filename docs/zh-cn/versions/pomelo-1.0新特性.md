## pomelo udpconnector提供

根据网友的要求，在pomelo 1.0中提供了udpconnector。在该udpconnector中，采用了pomelo之前hybridconnector提供的传输协议，包括握手、心跳及数据包的格式。该connector中通过客户端的ip和port来对客户端进行唯一标识，客户端的断开则是通过心跳超时进行判定。

使用方法：

```

// app configuration

app.configure('production|development', 'connector', function() {

	app.set('connectorConfig',
		{
			connector : pomelo.connectors.udpconnector,
			heartbeat : 3
		});
});

```

由于pomelo中提供的客户端还没有支持udp的，所以在1.0.0中提供一个node版本的udp client，配合通过pomelo init出来的demo使用，具体可以参考[udpclient](https://github.com/py8765/udpclient/tree/master)


## pomelo-rpc 负载均衡及容错机制

在pomelo 1.0版本中，pomelo-rpc提供了相关的负载均衡算法，开发者在调用rpc时，可以选择相应的路由算法，rpc框架会根据不同的配置在rpc客户端进行路由算法的选择，从而完成相应的rpc调用。提供的负载均衡算法包括：rr（ROUNDROBIN）, wrr(WEIGHT_ROUNDROBIN), la(LEAST_ACTIVE), ch(CONSISTENT_HASH)。

配置方法：

```

// rr

app.configure('production|development', 'connector', function() {

	app.set('proxyConfig',
		{
			routerType: 'rr'
		});
});

//基于权重的rr，在servers.json中对rpc目标服务器进行权重配置

app.configure('production|development', 'connector', function() {

	app.set('proxyConfig',
		{
			routerType: 'wrr'
		});
});

 "read": [
 
    {"id": "read-server-1", "host": "127.0.0.1", "port": 4150, "weight": 1},
    {"id": "read-server-2", "host": "127.0.0.1", "port": 4151, "weight": 5},
    {"id": "read-server-3", "host": "127.0.0.1", "port": 4152, "weight": 8}
 ]

//la

app.configure('production|development', 'connector', function() {

	app.set('proxyConfig',
		{
			routerType: 'la'
		});
});

//consistent_hash

app.configure('production|development', 'connector', function() {

	app.set('proxyConfig',
		{
			routerType: 'ch',
            replicas: '100',   //虚拟节点数量
            algorithm: 'md5',  //hash算法
			hashFieldIndex: 0 //根据rpc参数列表中的具体参数进行hash
		});
});

```

ps:　如果使用toServer('chat-server-1')这种指定rpc目标服务器的rpc调用则不会使用相关的负载均衡算法，如果是指定了routerType则之前在application对象中设置的route函数则无效；综合这三种方式的优先级是toServer > 指定routerType > 指定路由函数。

在新版本中,pomelo-rpc模块提供了相关的容错机制，包括failover，即失败自动切换，当出现失败，重试其它同类型服务器，这种模式通常用于读操作，但重试可能会带来更长延迟；failfast是快速失败，其策略为只发起一次rpc调用，失败后就立即将错误信息返回；failsafe则是一种安全策略，也是rpc默认采用的，即根据rpc的不同类型错误进行不同的处理策略，主要是发起连接重试和发送重试操作。

配置方法：

```

//快速失败

app.configure('production|development', 'connector', function() {

	app.set('proxyConfig',
		{
			failMode : 'failfast'
		});
});

//切换服务器

app.configure('production|development', 'connector', function() {

	app.set('proxyConfig',
		{
			failMode : 'failover'
		});
});

//安全策略

app.configure('production|development', 'connector', function() {

	app.set('proxyConfig',
		{
			failMode : 'failsafe',
			retryTimes:　3， //重试次数
			retryConnectTime：　5 * 1000 //重连间隔时间
		});
});


```

ps: rpc默认采用安全策略。


## pomelo 支持tls及wss

考虑到安全性方面的问题，pomelo 1.0版本中增加了对tls及wss协议的支持；在之前的pomelo版本中，提供了hybridconnector和sioconnector；对于sioconnector,其底层使用的是socket.io，socket.io提供了包括websocket和长轮询等几种传输方式，其中websocket默认采用的ws协议，现在pomelo支持wss协议，即用户可以在sioconnector中配置基于wss协议的websocket的通信方式；对于hybridconnector,提供包括原生socket的支持和websocket的支持，针对这两种方式，在pomelo 1.0中提供了两种连接的安全版本即tls和wss,用户可以在使用hybridconnector时，采用安全级别较高的tls或者wss。

使用方法：

支持tls客户端，pomelo现在提供的tls客户端有libpomelo.

```

app.configure('production|development', 'connector', function() {

  app.set('connectorConfig',

    {
      connector : pomelo.connectors.hybridconnector,
      heartbeat : 3,
      useDict : true,
      useProtobuf : true,
      ssl: {
      	key: fs.readFileSync('./keys/server.key'),
  	cert: fs.readFileSync('./keys/server.crt'),
      }
    });
});

```

支持wss客户端, pomelo提供的wss客户端有js客户端.

```

app.configure('production|development', 'connector', function() {

  app.set('connectorConfig',

    {
      connector : pomelo.connectors.hybridconnector,
      heartbeat : 3,
      useDict : true,
      useProtobuf : true,
      ssl: {
      	key: fs.readFileSync('./keys/server.key'),
  	cert: fs.readFileSync('./keys/server.crt'),
      }
    });
});

```

支持socket.io的wss客户端, pomelo提供的socket.io的wss客户端有js客户端.

```

app.configure('production|development', 'connector', function() {

  app.set('connectorConfig',

    {
      connector : pomelo.connectors.sioconnector,
      key: fs.readFileSync('./keys/server.key'),
  	  cert: fs.readFileSync('./keys/server.crt')
    });
});


```

在pomelo 1.0中增加了通过pomelo init 获取wss和socket.io的wss两种客户端及服务端的初始化项目，同时初始化的项目中提供了相应的密钥及证书。注意由于证书是和域名绑定的，所以在打开客户端的时候输入的ip地址为  https://127.0.0.1:3001


## pomelo 提供zookeeper集群管理

在pomelo之前版本中，提供了master服务器进行集群管理，对于中小型游戏项目及非大型分布式集群项目来说已经能够满足需求；对于大型的分布式项目的集群管理，zookeeper则是最佳选择。为了让pomelo在更广泛的领域适用，在1.0版本中pomelo提供了zookeeper的插件。

其主要原理是利用zookeeper的集群管理，使得pomelo集群中服务器信息保持一致，确保pomelo对外提供稳定的服务。具体实现则是master服务器启动后主动向zookeeper注册一个根节点，其它服务器启动后作为这个根节点的孩子注册到zookeeper,并将服务器相关信息写入，同时非master服务器监听该根节点的孩子节点变化情况；一旦孩子节点有变化，则其它节点重新从根节点或者当前集群中服务器信息。


使用方法：

```

   var zookeeper = require('pomelo-zookeeper-plugin');

   app.configure('production|development', function() {

    //关闭master集群管理功能
	app.set('masterConfig', {
		closeWatcher: true
	});
	app.set('monitorConfig', {
		closeWatcher: true
	});

	app.use(zookeeper, {
		zookeeper: {
			server: '127.0.0.1:2181',
			path: '/pomelo/servers',
			username: 'pomelo',
			password: 'pomelo'
		}
	});
 
  }

```

github地址： [pomelo-zookeeper-plugin](https://github.com/NetEase/pomelo-zookeeper-plugin)


## libpomelo 提供自动重连及SSL/TLS支持

libpomelo自动重连
===================

关于libpomelo的自动重连，目前使用的策略为一旦检测到读写socket错误或者数据包解析遇到非法包时，将重置连接，并发起自动重连，重连成功后抛出reconnect事件。
用户可以通过显式地调用pc_client_disconnect来断开连接，这样的话，将不会发起自动重连。

新增API:

- pc_client_t* pc_client_new_with_reconnect(int delay, int delay_max, int exp_backoff);
参数说明：
    delay - 这个参数是重连时，用来退避的延时值，单位为秒
    delay_max - 这个参数是重连时最大延时值，单位也为秒
    exp_backoff - 是否开启指数退避，非0时为开启指数退避，为0时不开启指数退避
返回值：
    成功，返回创建成功的实例
    失败，返回NULL

例如，如果delay为2s，delay_max为10s，exp_backoff为0，不开启指数退避，那么重连时的连接重试间隔将为
       2s， 4s， 6s， 8s， 10s， 10s， 10s......直到连接上为止

      如果delay为2s， delay_max为30s，exp_backoff为1， 开启指数退避，那么重连时的连接重试间隔将为
       2s， 4s， 8s， 16s， 30s， 30s.....直到连接上为止


- void pc_client_disconnect(pc_client_t* client);
参数说明:
     client 为pomelo client实例


libpomelo SSL/TLS支持
=======================

libpomelo目前计划支持SSL/TLS，使用OpenSSL。在使用TLS支持时，在编译时需开启宏 WITH_TLS。 **拟**增加API如下：

- int pc_client_lib_init();
  这个函数主要用来做一些全局SSL/TLS相关的以及库相关的初始化，此函数应该在所有其他库函数执行前调用一次，不用检查返回值，返回值永远为0。

- int pc_client_lib_cleanup();
  这个函数与pc_client_lib_init相对应，用来在应用关闭时，对整个库做一些全局化的清理工作，其应当是最后一个被调用的库函数，返回值永远为0

- int pc_client_set_tls_ca(pc_client_t* client, const char* cafile, const char* capath)
  这个函数用来设置SSL/TLS用到的ca信息.
参数说明：
  client - pomelo client实例.
  cafile,capath - 这两个参数用来指定ca信息，具体请参见SSL的调用[SSL_CTX_load_verify_locations](https://www.openssl.org/docs/ssl/SSL_CTX_load_verify_locations.html).
返回值:
  成功，返回值为0
  参数错误时，返回值为-1

- int pc_client_set_tls_cert(pc_client_t* client, const char* certfile, const char* keyfile, int (\*pw_callback)(char* buf, int size, int rwflag, void* userdata));
  这个函数是服务端要求对pc client进行验证时，用来设置客户端的证书以及私钥信息， 如果不需要双向验证，则不需要设置。

参数说明: 
  cerfile - 存放证书链的文件，PEM格式，具体请参见OpenSSL的调用[SSL_CTX_use_certificate_chain_file](https://www.openssl.org/docs/ssl/SSL_CTX_use_certificate.html)
  keyfile - 存放私钥的文件，PEM格式。
  pw_callback - 如果私钥文件使用口令加密的话，此参数会被回调用来传递解密私钥文件的口令，否则为NULL即可，具体参见 [SSL_CTX_set_default_passwd_cb](https://www.openssl.org/docs/ssl/SSL_CTX_set_default_passwd_cb.html#)

- int pc_client_set_tls_opts(pc_client_t* client, int enable_verify, const char* ciphers);
  这个参数是用来设置SSL的一些选项的，OpenSSL的提供的选项很多，这里仅提供了两处选项设置
参数说明：
  enable_verify - 是否开启服务端证书验证，将此值设置为0，将不验证服务端证书，设置为1将验证服务端证书，缺省时不对服务端证书进行验证
  ciphers - 一个字符串用来说明可用的加密算法，如果使用缺省值的话，只需设置为NULL即可

- int pc_client_set_tls_hostname_verify(pc_client_t* client, int (\*hostname_verify_cb)(pc_client_t* client, const char** names, int len));
  这个函数用来设置对服务端的hostname进行验证的回调的

参数说明： 
   hostname_verify_cb - 这个参数是一个回调用来验证服务器端证书中的SAN以及CN字段的，如果设置了此回调，那么当收到证书后，libpomelo会从证书中解析出SAN和CN，并调用此回调函数，此函数返回1表示验证通过，返回0表示验证失败，参数 names 和 len 是用来表示一个字符串数组，其内部存储着证书中解析出来的SAN和CN。对于names，用户在实现回调时，只需要访问其获取SAN和CN即可，不要对其进行其他的操作，libpomelo会负责其内存的申请和释放。


说明
=========================

关于自动重连和TLS支持，目前还处于实验阶段，其中自动重连已经可以测试，但是TLS实现尚未加完，在github上有auto-reconn分支和ssl分支，也欢迎各位把你修改后的或者更好的实现代码贡献出来, 谢谢。
关于libpomelo任何想法，无论是代码组织，还是具体的实现策略，欢迎大家互相交流。
  

## pomelo 提供自动扩展插件

在pomelo1.0里提供了一个服务器自动扩展的插件，其主要原理是监控某一类型的服务器，监控的指标现在暂时提供cpu和memory，当这一类型的服务器的某项监控指标超过之前设置的阈值时，服务器就自动扩展，扩展服务器的数量可以由用户进行配置。

使用方法：

```
//app.js配置方法

app.configure('production|development', 'master', function() {

	app.use(scale, {
		scale: {
			cpu: {
				chat: 5,
				interval: 10 * 1000,
				increasement: 1
			},
			memory: {
				connector: 5,
				interval: 15 * 1000,
				increasement: 1
		  },
		  backup: 'config/development/backupServers.json'
		}
	});
});

```


```
//backupServer.json配置

{
    "connector":[

             {"id":"backup-connector-server-1", "host":"127.0.0.1", "port":4053, "clientPort": 3053, "frontend": true},
             {"id":"backup-connector-server-2", "host":"127.0.0.1", "port":4054, "clientPort": 3054, "frontend": true},
             {"id":"backup-connector-server-3", "host":"127.0.0.1", "port":4055, "clientPort": 3055, "frontend": true}
         ],
        "chat":[
             {"id":"backup-chat-server-1", "host":"127.0.0.1", "port":6053},
             {"id":"backup-chat-server-2", "host":"127.0.0.1", "port":6054},
             {"id":"backup-chat-server-3", "host":"127.0.0.1", "port":6055}
        ]
}


```

配置参数说明：

现在监控指标包括cpu和memory两项，在每一个监控指标内可以有监控的服务器类型，例如chat：5，这样就表示chat类型的服务器的阈值为5%，当chat类型的服务器cpu的平均值超过5%后，系统将自动扩展服务器，服务器一次扩展的数量由increasement参数决定，例如increasement参数为1，则表示每次超过阈值后扩展1个服务器，扩展服务器的列表由用户指定，backup参数就是扩展的服务器列表；另外interval参数表示系统检测时间，单位是秒，例如interval: 15 * 1000表示系统每15秒检测一次相应的指标，如果超过该指标则进行相应的扩展。

github地址： [pomelo-scale-plugin](https://github.com/NetEase/pomelo-scale-plugin)


## 支持按环境目录配置

在pomelo1.0中支持按照目录结构进行配置相关的配置文件，在之前的版本中pomelo的配置文件如下图所示：

<img src='https://raw.githubusercontent.com/py8765/pomelo_images/master/dir.bmp'>


不同环境是根据具体配置文件里的key进行区分，例如：

```

// servers.json配置

{
    "development":{

        "connector":[
             {"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort": 3050, "frontend": true}
         ]
    },
    "production":{
           "connector":[
             {"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort": 3050, "frontend": true}
         ]
  }
}

```

在pomelo1.0支持根据目录进行配置，如下图所示：

<img src='https://raw.githubusercontent.com/py8765/pomelo_images/master/new_dir.bmp'>


config 目录下是根据环境进行文件配置， 在启动过程中选择不同的环境就会根据相应的环境名称目录加载该目录下的所有配置文件。例如 pomelo start env=online 这样就会加载config/online目录下的所有配置文件。默认会加载development下面的配置文件。在这种情况下，对应的servers.json就不需要根据环境配置，具体配置如下：

```

{

        "connector":[
             {"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort": 3050, "frontend": true}
         ]
}


```

PS: 默认还是会直接加载config目录下的配置文件，当config目录下面没有对应的文件系统将才会加载环境名称对应的目录下的配置文件；所以要使用根据环境名称目录进行配置时需要先将config目录下之前的配置文件删除。

## 支持pomelo-cli输入脚本

pomelo 1.0版本中，在pomelo-cli中增加了对脚本输入的支持，主要是方便通过pomelo-cli对不同服务器进行动态地查看相关信息，例如在前端服务器查看相应的session数量，某一个用户的session，或者某一个用户的地址信息等等。


使用方法：

```

pomelo-cli　　//进入pomelo-cli

use connector-server-1  //使用具体服务器

run app.get("sessionService").getSessionsCount()  //执行相应的命令


```

ps: 脚本默认app为每个服务器的application对象，考虑到通过application对象可以获取到这个服务器的所有信息，所以在这个执行脚本中只提供application对象。

## sioconnector支持低版本的IE浏览器

之前有很多网友提出希望sioconnector支持低版本的IE浏览器，因为之前sioconnector的通信方法使用了pomelo-protocol,在encode/decode过程中使用了一些低版本IE浏览器不支持的数据结构，在1.0正式版本的pomelo中提供了新的js客户端对IE低版本的客户端支持。客户端代码地址：[pomeloclient.js](https://github.com/pomelonode/pomeloclient)

使用方法：

在web-server中将之前的pomeloclient.js替换为pomeloclient_ie.js，在game-server中进行对app.js进行配置如下：

```

var encode = function(reqId, route, msg) {

    if (!!reqId) {
        return{
            id: reqId,
            body: msg
        };
    } else {
        var m = {
            route: route,
            body: msg
        };
        return JSON.stringify(m);
    }
};

var decode = function(data) {

    data = JSON.parse(data);
    return {
        id: data.id,
        route: data.route,
        body: data.msg
    };
};

//将对应的前端服务器进行配置
app.configure('production|development', 'connector|gate', function() {

        app.set('connectorConfig', {
        connector: pomelo.connectors.sioconnector,
        encode: encode,
        decode: decode
    });
});


```


## mqtt connector 支持

在1.0正式版本的pomelo中，发布了mqtt connector.考虑到pomelo本身的体系的兼容，mqtt connector实际上并未采用publish/subscribe的模式。因为mqtt本身协议具有精简、灵活等优势，比较适合移动设备使用，所以pomelo引入mqtt connector主要是使用其协议。


使用方法：

```

app.configure('production|development', 'connector', function() {

 app.set('connectorConfig', {

    connector: mqttconnector,
    publishRoute: 'connector.mqttHandler.publish',
    subscribeRoute: 'connector.mqttHandler.subscribe'
  });

});


```

其中publishRoute指的是客户端通过publish方法发到服务端的消息所导向的handler具体方法，例如publishRoute: 'connector.mqttHandler.publish'表示客户端的publish消息将导入connector的mqttHandler文件的publish方法，subscribeRoute与publishRoute类似。



## handler及remote的部分热更新支持

在1.0正式版本中，提供了handler和remote的热更新的部分支持。在pomelo1.0正式版中，提供对handler和remote两个目录中的文件进行监控，当有文件更新时，系统会重新load对应的handler和remote服务，这时当有新的handler请求和rpc请求进入时，系统就会自动执行更新后的服务。但是对于在非handler和remote目录的文件就不支持热更新。热更新开关默认是关闭状态，用户可以通过配置打开热更新服务。

使用方法：

```

  // handler 热更新开关 

	app.set('serverConfig',
	{
		reloadHandlers: true
	});


 // remote 热更新开关 
	app.set('remoteConfig',
	{
		reloadRemotes: true
	});


```

## 其它

在1.0版本中取消对toobusy和ndump模块的支持，但保留toobusy的filter,如果需要可以自己在项目中安装toobusy模块；对于ndump则采用node-heapdump取代，pomelo-cli暂时只支持对内存的dump，停止对cpu dump的支持。