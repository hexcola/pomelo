---
title: 术语解释
---

<!-- TOC -->

- [常见术语](#常见术语)
    - [gate 服务器](#gate-服务器)
    - [connector 服务器](#connector-服务器)
    - [应用逻辑服务器](#应用逻辑服务器)
    - [master 服务器](#master-服务器)
    - [rpc 调用](#rpc-调用)
    - [route,router](#routerouter)
    - [Session, FrontendSession, BackendSession， SessionService， BackendSessionService](#session-frontendsession-backendsession-sessionservice-backendsessionservice)
    - [Channel](#channel)
    - [request, response, notify, push](#request-response-notify-push)
    - [filter](#filter)
    - [handler](#handler)
    - [error handler](#error-handler)
    - [component](#component)
    - [admin client, monitor, master](#admin-client-monitor-master)
    - [admin module](#admin-module)
    - [plugin](#plugin)
- [小结](#小结)

<!-- /TOC -->

使用 pomelo 框架的话，有 pomelo 自己的术语，这里先对术语做一些简单的解释，给读者一个直观的概念，不至于看到相应术语时产生迷惑。

## 常见术语

### gate 服务器

一个应用的 gate 服务器，一般不参与 rpc 调用，也就是说其配置项里可以没有 port 字段，仅仅有 clientPort 字段，它的作用是做前端的负载均衡。客户端往往首先向 gate 服务器发出请求，gate 会给客户端分配具体的 connector 服务器。具体的分配策略一般是根据客户端的某一个 key 做 hash 得到 connector 的 id，这样就可以实现各个 connector 服务器的负载均衡。

### connector 服务器

connector 服务器接收客户端的连接请求，创建与客户端的连接，维护客户端的 session 信息。同时，接收客户端对后端服务器的请求，按照用户配置的路由策略，将请求路由给具体的后端服务器。当后端服务器处理完请求或者需要给客户端推送消息的时候，connector 服务器同样会扮演一个中间角色，完成对客户端的消息发送。connector 服务器会同时拥有 clientPort 和 port，其中 clientPort 用来监听客户端的连接，port 端口用来给后端提供服务。

### 应用逻辑服务器

gate 服务器和 connector 服务器又都被称作前端服务器，应用逻辑服务器是后端服务器，它完成实际的应用逻辑，提供服务给客户端，当然客户端的请求是通过前端服务器路由过来的。后端服务器之间也会通过 rpc 调用而有相互之间的交互。由于后端服务器不会跟客户端直接有连接，因此后端服务器只需监听它提供服务的端口即可。

### master 服务器

master 服务器加载配置文件，通过读取配置文件，启动所配置的服务器集群，并对所有服务器进行管理。

### rpc 调用

pomelo 中使用 rpc 调用进行进程间通信，在 pomelo 中 rpc 调用分为两大类，使用 namespace 进行区分，namespace 为 sys 的为系统 rpc 调用，它对用户来说是透明的，目前 pomelo 中系统 rpc 调用有：

- 后端服务器向前端服务器请求 session 信息
- 后端服务器通过 channel 推送消息时对前端服务器发起的 rpc 调用
- 前端服务器将用户请求路由给后端服务器时也是 sys rpc 调用

除了系统 rpc 调用外，其余的由用户自定义的 rpc 调用属于 user namespace 的 rpc 调用，需要用户自己完成 rpc 服务端 remote 的 handle 代码，并由 rpc 客户端显式地发起调用

### route,router

route 用来标识一个具体服务或者客户端接受服务端推送消息的位置，对服务端来说，其形式一般是`<ServerType>.<HandlerName>.<MethodName>`, 例如 "`chat.chatHandler.send`", chat 就是服务器类型，chatHandler 是 chat 服务器中定义的一个 Handler，send 则为这个 Handler 中的一个 handle 方法。对客户端来说，其路由一般形式为 onXXX，当服务端推送消息时，客户端会有相应的回调。

一般来说具体的同类型应用服务器都会有多个，当客户端请求到达后，前端服务器会将用户客户端请求派发到后端服务器，这种派发需要一个路由函数 router，可以粗略地认为 router 就是根据用户的 session 以及其请求内容，做一些运算后，将其映射到一个具体的应用服务器 id。可以通过 application 的 route 调用给某一类型的服务器配置其 router。如果不配置的话，pomelo 框架会使用一个默认的 router。pomelo 默认的路由函数是使用session 里面的 uid 字段，计算 uid 字段的 crc32 校验码，然后用这个校验码作为 key，跟同类应用服务器数目取余，得到要路由到的服务器编号。

> 注意这里有一个陷阱，就是如果 session 没有绑定 uid 的话，此时 uid 字段为 undefined，可能会造成所有的请求都路由到同一台服务器。所以在实际开发中还是需要自己来配置router。

### Session, FrontendSession, BackendSession， SessionService， BackendSessionService

在 pomelo 框架中，有这三个 session 的概念，同时又有两个 service： `SessionService` 和`BackendSessionService`，也是最令人迷惑的地方，这里尝试给出一些说明，让你的理解更清晰一些：
Session 指的是一个客户端连接的抽象，它的大致字段如下：

```js
{
    id : <session id> // readonly
    frontendId : <frontend server id> // readonly
    uid : <bound uid> // readonly
    settings : <key-value map> // read and write  
    __socket__ : <raw_socket>
    __state__ : <session state>

    // ...
}
```

- `id` 是这个 session 的 id，是全局唯一的，一般使用自增的方式来生成;
- `frontendId` 是维护这个 session 的前端服务器的id；
- `uid` 是这个 session 所绑定的用户 id;
- `__socket__` 是底层原生 socket 的引用;
- `__state__` 用来指明当前 session 的生命周期状态。
- `settings` 维护一个 `key-value map`，用来描述 session 的一些自定义属性，比如聊天应用中的房间号就可以看作是 session 的一个自定义属性。

从上面的分析看，一个 session 一旦建立，那么 `id`， `frontendId`，`__socket__`, `__state__`, `uid` 都是确定的，都应该是只可读不可写的。而 `settings` 也不应该被随意的修改。

因此，在前端服务器中，引入了 `FrontendSession`, 可以把它看作是一个内部 `session` 在前端服务器中的傀儡，`FrontendSession` 的字段大致如下:

```js
{
    id : <session id> // readonly
    frontendId : <frontend server id> // readonly
    uid : <bound uid> // readonly
    settings : <key-value map> // read and write  
}
```

其作用：

- 通过 FrontendSession 可以对 settings 字段进行设置值，然后通过调用 FrontendSession 的 push 方法，将设置的 settings 的值同步到原始 session 中;
- 通过 FrontendSession 的 bind 调用，还可以给 session 绑定 uid;
- 当然也可以通过 FrontendSession 访问 session 的只读字段，不过对 FrontendSession 中与 session 中相同的只读字段的修改并不会反映到原始的 session 中。

SessionService 维护所有的原始的 session 信息,包括不可访问的字段，绑定的uid以及用户自定义的字段。

下面再说 BackendSession，与 FrontendSession 类似，BackendSession 是用于后端服务器的，可以看作是原始 session 的代理，其数据字段跟 FrontendSession 基本一致。

BackendSession 是由 BackendSessionService 创建并维护的，在后端服务器接收到请求后，由BackendSessionService 根据前端服务器 rpc 的参数，进行创建。对 BackendSessionService 的每一次方法调用实际上都会生成一个远程调用，比如通过一个 sid 获取其 BackendSession。同样，对于 BackendSession 中字段的修改也不会反映到原始的 session中，不过与 FrontendSession 一样，BackendSession 也有 push，bind，unbind 调用，它们的作用与 FrontendSession 的一样，都是用来修改原始session 中的 settings 字段或者 绑定/解绑 uid 的，不同的是 BackendSession 的这些调用实际上都是名字空间为 sys 的远程调用。

### Channel

channel 可以看作是一个玩家 id 的容器，主要用于需要广播推送消息的场景。可以把某个玩家加入到一个 Channel 中，当对这个 Channel 推送消息的时候，所有加入到这个 Channel 的玩家都会收到推送过来的消息。一个玩家的 id 可能会被加入到多个 Channel 中，这样玩家就会收到其加入的 Channel 推送过来的消息。需要注意的是 Channel 都是服务器本地的，应用服务器 A 和 B 并不会共享 Channel，也就是说在服务器 A 上创建的Channel，只能由服务器 A 才能给它推送消息。

### request, response, notify, push

pomelo 中有四种消息类型的消息，分别是 request，response，notify 和 push，客户端发起 request 到服务器端，服务器端处理后会给其返回响应r esponse; notify 是客户端发给服务端的通知，也就是不需要服务端给予回复的请求; push 是服务端主动给客户端推送消息的类型。在后面的叙述中，将会使用这些术语而不再作解释。


### filter

filter 分为 before 和 after 两类，每类 filter 都可以注册多个，形成一个 filter 链，所有的客户端请求都会经过 filter 链进行一些处理。before filter 会对请求做一些前置处理，如：检查当前玩家是否已登录，打印统计日志等。after filter 是进行请求后置处理的地方，如：释放请求上下文的资源，记录请求总耗时等。after filter 中不应该再出现修改响应内容的代码，因为在进入 after filter 前响应就已经被发送给客户端。

### handler

handler 是实现具体业务逻辑的地方，在请求处理流程中，它位于 before filter 和 after filter 之间，handler 的接口声明如下：

```js
handler.methodName = function(msg, session, next) {
  // ...
}
```

参数含义与 before filter 类似。handler 处理完毕后，如有需要返回给客户端的响应，可以将返回结果封装成js 对象，通过 `next` 传递给后面流程。

### error handler

error handler 是一个处理全局异常的地方，可以在 error handler 中对处理流程中发生的异常进行集中处理，如：统计错误信息，组织异常响应结果等。error handler 函数是可选的，如果需要可以通过

```js
app.set('errorHandler', handleFunc);
```

来向 pomelo 框架进行注册，函数声明如下：

```js
errorHandler = function(err, msg, resp, session, next) {
  // ...
}
```

其中，err 是前面流程中发生的异常；resp 是前面流程传递过来，需要返回给客户端的响应信息。其他参数与前面的handler一样。

### component

pomelo 框架是由一些松散耦合的component组成的，每个 component 完成一些功能。整个 pomelo 框架可以看作是一个 component 容器，完成 component 的加载以及生命周期管理。pomelo 的核心功能都是由 component 完成的，每个 component 往往有 start，afterStart，stop 等调用，用来完成生命周期管理。

### admin client, monitor, master

在对 pomelo 服务器进行管理的时候，有三个概念 admin client， monitor， master。

- monitor 运行在各个应用服务器中，它会向 master 注册自己，向 master 上报其服务器的信息，当服务器群有变化时，接收 master 推送来的变化消息，更新其服务器上下文。

- master 运行在应用服务器中，它会收集整个服务器群的信息，有变化时会将变化推送到各个 monitor；同时，master 还接受 admin client 的请求，按照 client 发出的命令，执行对应的操作，如查询整个服务器群的状态，增加一个服务器等。

- client 独立运行自己的进程，它会发起到 master 的连接，然后通过对 master 发出请求或者命令，来管理整个服务器群。目前工具 [pomleo-cli](https://github.com/NetEase/pomelo-cli)就是这样的一个客户端。

### admin module

在 pomelo 中，module 特指服务器监控管理模块，与 component 类似，不过在 module 中实现的是监控逻辑，比如收集进程状态等。用户在使用时，可以通过 `application` 的 `registerAdmin` 注册管理模块，实现用户自己定制的监控管理功能。每一个module中都会定义下面四种回调函数，不过都是可选的：
- masterHandler(agent, msg, cb) 当有应用服务器给master发监控数据时，这个回调函数会由master进程进行回调，完成应用服务器的消息处理;
- monitorHandler(agent, msg, cb) 当有master请求应用服务器的一些监控信息时，由应用服务器进行回调，完成对master请求的处理;
- clientHandler(agent, msg, cb）当由管理客户端向master请求服务器群信息时，由master进程进行回调处理客户端的请求。
* start(cb) 当admin module，注册加载完成后，这个回调会被执行，在这里可以做一些初始化工作。

### plugin

plugin 是 pomelo 0.6 加入的全新的扩展机制，一个 plugin 由多个 component 以及一些事件响应处理器组成。它提供了一种很灵活的机制来扩展 pomelo。不仅可以提供 component 的功能，还可以对整个框架的全局事件作出响应处理。

## 小结

上面简要地介绍了 pomelo 中的一些术语，因为在下面的例子中，会涉及到这些术语，不至于当出现这些术语时一头雾水。下面我们就正式进入我们的例子，[获取源码并安装我们的例子应用](chat源码下载与安装 "chat源码下载与安装")。

