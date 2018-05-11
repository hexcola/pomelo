---
title: 压缩
---

<!-- TOC -->

- [route 压缩](#route-压缩)
    - [chat 中使用](#chat-中使用)
    - [小结](#小结)
- [protobuf 压缩](#protobuf-压缩)
    - [chat 中使用](#chat-中使用-1)
    - [小结](#小结-1)

<!-- /TOC -->

# route 压缩

在实际编程中，网络带宽的有效数据负载率是一个值得考虑的问题。对于移动客户端来说，网络资源往往不是很丰富，为了尽可能地节省网络资源，往往需要尽大可能地增加数据包的有效数据率。

以我们的聊天应用为例，当客户端发起聊天时，需要指定处理其请求的服务器的路由信息，示例如下：

```js

pomelo.request('chat.chatHandler.send', 
  // ...
);

```

这个路由信息指出，处理这个请求的应该是 chat 服务器的 chatHandler 的 send 方法。当服务器给客户端推送消息的时候，同样也需要指明客户端的路由信息，在例子聊天应用中有 onAdd，onLeave 等。考虑当用户发起聊天的信息很短的时候，比如用户仅仅发了一个字，而我们在传输的时候一样要加上一个完整的路由信息，这样将造成实际传输中，有效数据率极低，网络资源被大量的额外信息浪费。最直接的想法就是缩短路由信息，对服务端的路由信息来说，由于当服务器确定后，其路由信息就确定了，对于客户端来说，虽然可以起很短的名字，但是很容易造成程序不可读。

针对这种情况，pomelo 提供了基于字典的路由信息压缩。

- 对于服务端，pomelo会扫描所有的 Handler 信息
- 对于客户端，用户需要在 `config/dictionary.json` 中声明所有客户端使用的路由。

通过这两种方式，pomelo 会拿到所有的客户端和服务端的路由信息，然后将每一个路由信息都映射为一个小整数，从1开始映射，累加。目前 pomelo 的路由信息压缩仅仅支持使用 hybridconnector 的方式，使用 sioconnector 的方式，暂不支持。在 hybridconnector 的实现中，如果使用了路由信息压缩，在客户端与服务器建立连接的握手过程中，服务器会将整个字典传给客户端，这样在以后的通信中，对于路由信息，将全部使用定义的小整数进行标记，大大地减少了额外信息开销。

## chat 中使用

下面我们就将 route 压缩用到我们的 chat 示例中，具体的代码在分支 `tutorial-dict` 中，使用下面命令切换分支：

```
$ git checkout tutorial-dict
```

首先看看客户端有哪些路由信息，我们把它放到 `config/dictionary.json` 里:

```js

// dictionary.json
[
  'onChat',
  'onAdd',
  'onLeave'
]

```

然后我们在 `connector` 配置选项里面增加 `useDict` 设置为 `true`。

```js

app.configure('production|development','connector', function() {
  app.set('connectorConfig', {
    connector: pomelo.connectors.hybridconnector,
    heartbeat: 3,
    useDict: true // enable dict
  });
});

app.configure('production|development','gate', function() {
  app.set('connectorConfig', {
    connector: pomelo.connectors.hybridconnector,
    useDict: true // enable dict
  });
});

```

- 好了，现在我们就已经开启了 pomelo 的路由压缩，现在的所有的数据包的路由信息都变成小整数了。
- 对于 dictionary 中添加的客户端路由，会使用路由压缩。如果有客户端的推送路由没有加入到 dictionary 中，会怎么样呢？不用怕，对于在 dictionary 中找不到的路由信息，pomelo 还是会使用原来不压缩的路由。

## 小结

到目前位置，我们客户端与服务器之间使用的消息传输格式一直都是 json。实际上，虽然 json 很方便，但是由于其还带了一些字段信息，在客户端和服务端数据包格式统一的情况下，这些字段信息是可以省略的，可以直接传输具体的消息，也就是说不再以字符串作为通信格式了，直接发送有效的二进制数据将会更好地减少额外开销，下面我们会使用 pomelo 提供的[protobuf 实现](Protobuf压缩 "Protobuf压缩")应用到我们的聊天应用中，以使得我们的应用更完善。

----

# protobuf 压缩

上面我们使用了 dictionary 的方式对聊天应用中的路由信息进行了压缩，减少了很多通信中的额外开销。在这里，我们将使用 pomelo 提供的 protobuf 实现完成通信消息的基于 protobuf 的压缩。protobuf 是 google 提出的数据交换格式，关于 protobuf 的更多信息请参阅[这里](https://github.com/google/protobuf)。

原始的 protobuf，首先需要定义一个 `.proto` 文件，然后调用 `protoc` 进行编译，根据不同的宿主语言，生成源码，然后将生成的源码应用到具体使用 protobuf 的应用中。这种使用方式比较笨重，因为涉及到了静态编译，应用程序无法在运行时动态地使用，一旦数据格式有变，就需要修改 proto，编译，重新生成源码。

pomelo 的 protobuf 实现，借助了 javascript 的动态性，使得应用程序可以在运行时解析 proto 文件，不需要进行 proto 文件的编译。pomelo 的实现中，为了更方便地解析 proto 文件，使用了 json 格式，与原生的proto 文件语法是相通的，但是是不相同的。用户定义好客户端以及服务端的通信所需要的信息格式的 proto 文件，服务端的 proto 配置放在 `config/serverProtos.json` 中，客户端的 proto 配置放在 `config/clientProtos.json`。如果在其配置文件里，配置了所有类型的 `proto` 信息，那么在通信过程中，将会全部使用二进制的方式对消息进行编码; 如果没有定义某一类消息相应的 `proto` ，pomelo 还是会使用初始的json 格式对消息进行编码。

## chat 中使用

下面将 pomelo-protobuf 应用到我们的聊天应用中，具体的代码在分支 `tutorial-protobuf` 中，使用下面命令切换分支：

```bash
$ git checkout tutorial-protobuf
```

1.  首先提取所有的数据格式，分为客户端使用的数据格式以及服务器端使用的数据格式，如下：

```js

// clientProtos.json
{
  "chat.chatHandler.send": {
    "required string rid": 1,
    "required string content": 2,
    "required string from": 3,
    "required string target": 4
  },

  "connector.entryHandler.enter": {
    "required string username": 1,
    "required string rid": 2
  },

  "gate.gateHandler.queryEntry": {
    "required string uid": 1
  }
}

// serverProtos.json
{
  "onChat": {
    "required string msg": 1,
    "required string from": 2,
    "required string target": 3
  },

  "onLeave": {
    "required string user": 1
  },

  "onAdd": {
    "required string user": 1
  }
}

```

2.  然后将这两个配置文件分别命名为 `clientProtos.json` 和 `serverProtos.json` 中，并将这两个配置文件都放到 `config` 目录下;

3.  在我们的程序中开启 protobuf，在 app.js 的配置中，增加 protobuf 使用，在配置 connector 的时候，加入 useProtobuf:


```js
app.configure('production|development', 'connector',  function() {
  app.set('connectorConfig', {
    connector: pomelo.connectors.hybridconnector,
    heartbeat: 3,
    useDict: true,
    useProtobuf: true //enable useProtobuf
  });
});

app.configure('production|development', 'gate', function(){
	app.set('connectorConfig', {
			connector : pomelo.connectors.hybridconnector,
			useDict: true,
      useProtobuf: true //enable useProtobuf
		});
});

```

这样，我们对我们的聊天应用进行了 protobuf 的压缩。当然，我们这里仅仅是为了示例，实际上，对于 onAdd 以及 onLeave 这样的，数据包本身就很小，而且又是字符串，对其使用 proto 压缩的效果不大，完全没必要进行使用proto 压缩，而且使用 protobuf 压缩会造成编解码的效率开销，得不偿失。实际运用中，还是需要根据实际情况进行合理的选择，更多时候我们是在消息的压缩率和编解码的开销中达到一个平衡。

对于 proto 文件里面没有配置的通信数据类型，pomelo 依然会使用原始的基于 json 的数据通信格式。

## 小结

到这里为止，我们已经实现了一个功能基本完善的聊天应用了，我们使用了 pomelo 提供的 filter 机制，基于dict 的 route 压缩和基于 protobuf 的消息压缩。下面将给聊天应用增加一些纯属 “画蛇添足” 的一些功能，目的是为了继续展示 pomelo 的特性。下一步，[给聊天应用增加一个rpc调用](增加rpc调用 "rpc调用")。