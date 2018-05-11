---
title: router 与服务器扩充
---

<!-- TOC -->

- [配置修改](#配置修改)
- [router配置](#router配置)
- [一些说明](#一些说明)
- [小结](#小结)

<!-- /TOC -->

当我们的应用只有很少人用的时候，往往只需要一台服务器就可以支撑。但是随着用户的增加，一台服务器可能就无法承受同一时刻巨大的访问量，这需要我们对服务器进行伸缩扩充。

多服务器版本的聊天应用在分支 `tutorial-multi-server` 上，你需要执行如下命令来切换到多服务器分支上：

```bash
$ git checkout tutorial-multi-server
```

## 配置修改

在 pomelo 中，对服务器的扩充非常简单，只需要修改一下配置文件，多添几台服务器配置就行了，对于我们的聊天例子来说，如下下面所示，在config/servers.json中的配置：

```json
{
 "development":{
      "connector":[
           {"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort": 3050, "frontend": true},
           {"id":"connector-server-2", "host":"127.0.0.1", "port":4051, "clientPort": 3051, "frontend": true},
           {"id":"connector-server-3", "host":"127.0.0.1", "port":4052, "clientPort": 3052, "frontend": true}
       ],
      "chat":[
           {"id":"chat-server-1", "host":"127.0.0.1", "port":6050},
           {"id":"chat-server-2", "host":"127.0.0.1", "port":6051},
           {"id":"chat-server-3", "host":"127.0.0.1", "port":6052}
      ],
      "gate":[
       {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true}
    ]
  },
  "production":{
    "connector":[
      {"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort": 3050, "frontend": true},
      {"id":"connector-server-2", "host":"127.0.0.1", "port":4051, "clientPort": 3051, "frontend": true},
      {"id":"connector-server-3", "host":"127.0.0.1", "port":4052, "clientPort": 3052, "frontend": true}
    ],
    "chat":[
      {"id":"chat-server-1", "host":"127.0.0.1", "port":6050},
      {"id":"chat-server-2", "host":"127.0.0.1", "port":6051},
      {"id":"chat-server-3", "host":"127.0.0.1", "port":6052}
    ],
    "gate":[
      {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true}
    ]
  }
}

```

## router配置

与前面的每个服务器类型仅有一台服务器的例子相比，这里我们的 connector 和 chat 都具有多台服务器。因此需要考虑对用户请求的服务器分配问题。

对于 gate 服务器来说，在前面的例子中，由于只有一个 connector 服务器，所以直接返回仅有的一个服务器就行了，而这里有多台服务器，所以需要从中选择一个服务器的信息进行返回，这里我们增加了一个工具函数 dispatch,它完成具体的分配运算，他使用用户的 uid 的 crc32 的校验码与 connector 服务器的个数取余，从而得到一个connector 服务器，大致代码如下:

```js
// util/dispatcher.js
module.exports.dispatch = function(key, list) {
    var index = Math.abs(crc.crc32(key)) % list.length;
      return list[index];
};

// gateHandler.js
handler.queryEntry = function(msg, session, next) {
  // ...

  // get all connectors
	var connectors = this.app.getServersByType('connector');

  // ...
  
  var res = dispatcher.dispatch(uid, connectors); // select a connector from all the connectors
  // do something with res
};

```

当客户端请求到来时，因为有多台 chat 服务器，需要选择由哪台 chat 服务器来服务，也就是前端服务器把这个客户端请求路由到哪个后端服务器上。配置路由使用 application 的 route 调用，这里我们也使用了前面提到的工具函数 dispatch，使用同样的服务器分配策略，示例如下：

```js
// app.js

// route definition for chat server
var chatRoute = function(session, msg, app, cb) {
  var chatServers = app.getServersByType('chat');

	if(!chatServers || chatServers.length === 0) {
		cb(new Error('can not find chat servers.'));
		return;
	}

	var res = dispatcher.dispatch(session.get('rid'), chatServers);

	cb(null, res.id);
};

app.configure('production|development', function() {
  app.route('chat', chatRoute);
});

```

其中 chatRoute 就是路由函数，他接受四个参数，返回一个其选择的后端服务器 id，四个参数中，第一个是专门用作路由计算的参数，前端服务器路由请求给后端服务器发 rpc 调用时，会使用 session 作为计算路由的参数，但是当用户自定定义 rpc 的时候，用户完全可以自己定义这个参数的含义，当然也可以使用 session。第二个参数 msg 描述了当前 rpc 调用的所有信息，包括调用的服务器类型，服务器名字，具体的调用方法等信息。第三个参数是一个上下文变量，一般情况下会由 app 来充当，第四个是一个获得到后端服务器 id 后的回调函数。

## 一些说明

- 通过修改服务器的配置文件，并增加具体的路由选择配置，就可以很轻易地实现服务器的扩充。 对于我们这个应用来说，如果我们还想继续扩充我们的服务器，那么只需在 servers.json 里面继续增加服务器的配置就行了。

- 如果我们一开始实现时就考虑以后的扩充，实现所有的路由选择函数的话，那么以后当需要扩充服务器的时候，只需要在 servers.json 里面增加相应的配置就行了。

- 关于客户端请求的路由，你可能会问，为什么我们在单服务器的那个例子里，没有给 chat 定义 router。那是因为如果我们不定义 router 的话，pomelo 会使用一个默认的 router 完成路由，因为只有一台 chat 服务器，那么 pomelo 总会把所有的对其的请求路由给这个服务器，所以，我们在前一个例子中，就省略掉了 chat 的路由配置。

- 实际应用中，我们一般都要自己实现 router,而不使用 pomelo 默认的。使用多台服务器要考虑负载平衡，同时要尽量使得服务器的服务是无状态的。我们 chat 的例子中，定义的 router 就是使用了用户的 rid 的 crc 校验码作为键值对当前的所有 chat 服务器个数做了一个简单的 hash 运算，以使得所有的 chat 服务器的负载尽可能平衡。

## 小结

在这部分，我们实现了对服务器的扩充，它是如此的简单，只需要修改相应的服务器配置即可。下面我们将尝试使用pomelo 的 [filter 机制](增加filter "增加filter") 继续完善我们的聊天应用。