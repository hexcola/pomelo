# Pomelo0.5新特性

在0.5版本中，主要增强了Pomelo高可用的特性，包括master服务器的高可用和其它服务器的可配置自动重启，另外还提供一个全局的globalChannel和服务器进程与cpu绑定的功能。

## master 高可用
Pomelo 虽然是分布式服务器框架，但是作为全局控制器的master节点却是唯一的。为了提高整体服务的可用性，我们0.5版中引入了基于Zookeeper的master高可用功能。
Master高可用是通过Zookeepr 实现的，因此如果要使用这一功能，需要第三方Zookeeper服务的支持。而Master高可用依赖于Zookpeer服务，因此建议作为第三方服务的Zookeeper本身要实现高可用。
### 实现原理
master高可用采用多机热备技术，采用主Master+多个备份Master节点的方式实现高可用。当主master宕机时，某一个从master会自动接管主master的工作，并与其他服务器建立连接，继续提供服务，这一过程全部是自动进行的。
### 使用说明
要开启Master高可用，只需要在app.js中加入以下配置：
```
app.configure('production|development', function(){
	app.enable('masterHA');
	app.set('masterHAConfig',
		{
			server : '127.0.0.1:2181',
			path : '/pomelo/master'
		});
});
```
app.enable('masterHA')表示开启master ha功能，这时服务器会加载zookeeper客户端，并尝试连接zookeeper默认端口“127.0.0.1:2181"。用户可以通过app.set('masterHAConfig'来加载自定义zookpeeper配置。server表示zookeeper服务器，path表示使用的zookeeper节点。该节点必须是zookeeper上已经存在并可以正常访问的节点。

现在有两种方式启动从Master节点：
* 第一种方式是在加入高可用配置后修改/config/master.json，采用与主master不同的Ip和端口，然后采用正常方式启动。这时，如果有主master启动的话，Pomelo会自动识别出主Master的存在，在启动时只会启动一个master备份节点，而不会重复启动其他服务。
* 第二种方式是使用命令行工具，直接启动一个独立的master节点。

## globalChannel

globalChannel是提供全局的channel服务，其默认实现是通过redis将相关信息存储，开发者可以根据自身需求开发其它实现;Pomelo原有的channelService只能在具体某个服务器中创建channel，这种channel只能存储该服务器的用户信息，而globalChannelService则可以创建全局的globalChannel，所有服务器的用户信息都可以通过globalChannel进行存储。

### 使用说明

globalChannel默认不加载，需要使用只需要在app.js中进行配置即可,参考配置如下(开启redis-server)。

```
app.configure('production|development', function(){
	app.set('globalChannelConfig',
		{
			host: '127.0.0.1',
		        port: 6379,
                        //optional
                        channelManager: mysqlManager
		});
});
var mysqlManager = function() {
// necessary methods (refer to redisGlobalChannelManager.js)
}
```

需要使用只需要从application中获取，即app.get('globalChannelService')；如果需要自己配置manager的实现，只需要在globalChannelConfig中配置channelManager,并参考Pomelo默认的redis的实现完成相应的方法就可以配置自己的globalChannel;具体的接口可以参考[Pomelo的API说明文档](http://pomelo.netease.com/api.html)。

## 服务器自动重启

根据网友的需求，在Pomelo0.5版本中增加了服务器（非master）自动重启的功能，服务器的自动重启是以服务器与master服务器的连接状态为判断依据，即当服务器与master服务器断开后触发该服务器的重新启动。建议只对无状态的服务器配置自动重启，这样能够保证服务器重启后不影响原系统的运行。默认情况下服务器不会自动重启，如果需要开启自动重启功能需要在servers.json中进行配置auto-restart，具体配置如下：

```
{
    "development":{
        "connector":[
            {"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort": 3050, "frontend": true}
         ]
        "chat":[
           {"id":"chat-server-1", "host":"127.0.0.1", "port":6050, "auto-restart": true}
        ]
        "gate":[
	   {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true}
	]
    }
}
```

## 服务器绑定CPU

为了更加充分的利用服务器的CPU，Pomelo在0.5版本中增加了服务器进程与指定CPU进行绑定，该功能限于linux系统的多核服务器，如果需要将服务器与具体CPU进行绑定，只需要在servers.json中进行配置，具体配置如下：

```
{
    "development":{
        "connector":[
             {"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort": 3050, "frontend": true, "cpu": 2}
         ]
        "chat":[
             {"id":"chat-server-1", "host":"127.0.0.1", "port":6050, "cpu": 1}
        ] 
        "gate":[
	     {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true, "cpu": 3}
	]
     }
}

```

## 自定义服务器关闭前事件

为了让开发者能够在服务器关闭前自定义处理事件，在Pomelo0.5版本中增加了这种服务器生命周期的自定义事件。开发者可以通过application对象的beforeStopHook方法添加自定义事件，示例代码如下：

```
app.configure('production|development', 'connector', function() {
          var fun = function(app, cb){
            //do something
            cb();
          }
          app.beforeStopHook(fun);
});
```