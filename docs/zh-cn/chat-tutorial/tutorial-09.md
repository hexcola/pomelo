---
title: 增加一个admin-module
---

<!-- TOC -->

- [chat 中使用](#chat-中使用)
- [一些说明](#一些说明)
- [小结](#小结)

<!-- /TOC -->

一个 pomelo 的应用，一般是由一个服务器群来支持，对于这些应用服务器的管理以及监控就显得尤为重要。比如监控这些应用服务器的进程状态，系统状态，杀死某个服务器等等。

对服务器的监控和管理有三个主体： master，monitor，client。服务器的管理和监控由 master 服务器加载的 master component 和普通的应用服务器加载的 monitor component，还有服务器管理客户端共同完成，下面的叙述中将不加区分地使用 monitor 与应用服务器，master 与 master 服务器。

master 负责收集所有服务器的信息，下发对服务器的操作指令。monitor 负责上报服务器状态，并对 master 的命令作出反应。client 是第三方监视的客户端，它注册到 master上，通过给master 发请求获得服务器群信息，或者给 master 发指令，管理操作应用服务器群。pomelo 中内建实现并使用了 console 和 watchdog 这两个 **admin module**，它们是 pomelo 核心的一部分,这里不再详述。

由于对于具体的应用来说，需要监控和管理的信息也是各不相同的，因此，pomelo 并没有实现固定的监控模块，而是提供了一个可插拔的监控框架机制，用户只需要定义一个监控模块所需要的回调方法，并完成相应的配置即可。

一组相关的供不同主体调用的回调函数构成一个 admin module，一个 admin module 中一般包括四个回调方法，`monitorHandler`，`masterHandler`，`clientHandler`, `start`。其中 `monitorHandler` 是 `monitor` 收到 master 的请求或者通知时由 monitor 回调，masterHandler 是 master 收到 monitor 的请求或者通知时回调，clientHandler 是master 收到 client 的请求或通知时回调的, start 是当admin module加载完成后，用来执行一些初始化监控时调用。

为了演示 admin module 的用法，我们将给聊天应用增加一个监控模块，我们让 monitor 每隔5秒钟向 master 上报一下自己的当前时间。当然，上报时间没有太多的实际意义，不过为了保持示例的简单化，选择上报时间还是可取的。实际使用中，可以上报任何信息，使用方式都是与上报时间的方式是一样的，这里使用上报时间仅仅是为了使得示例尽可能简单，更容易抓住如何使用 admin module。

## chat 中使用

下面我们将给我们的聊天应用增加一个监控管理模块，具体的代码在分支 `tutorial-admin-module`上，使用如下命令切换分支：

```bash
$ git checkout tutorial-admin-module
```

1. 首先，我们在 app 目录下建立文件 `modules/timeReport.js`, 在其中定义`monitorHandler`，`masterHandler` 和 `clientHandler`，代码如下：

```js

module.exports = function(opts) {
    return new Module(opts);
}

var moduleId = "timeReport";
module.exports.moduleId = moduleId;

var Module = function(opts) {
    this.app = opts.app;
    this.type = opts.type || 'pull';
    this.interval = opts.interval || 5;
}

Module.prototype.monitorHandler = function(agent, msg, cb) {
    console.log(this.app.getServerId() + '  ' + msg);
    var serverId = agent.id;
    var time = new Date().toString();

    agent.notify(moduleId, {serverId: serverId, time: time});
};

Module.prototype.masterHandler = function(agent, msg) {
    if (!msg) {
      var testMsg = 'testMsg';
      agent.notifyAll(moduleId, testMsg);
      return;
    }

    console.log(msg);
    var timeData = agent.get(moduleId);
    if (!timeData) {
        timeData = {};
        agent.set(moduleId, timeData);
    }
    timeData[msg.serverId] = msg.time;
};


Module.prototype.clientHandler = function(agent, msg, cb) {
    cb(null, agent.get(moduleId));
}

```

2.  这里我们没有定义 start 回调，因为我们这里用不到。在定义完上面的 admin module 后，需要将其注册到我们的应用中，使用 `Application.registerAdmin` 调用，在 `app.js` 中增加如下代码：

```js

var timeReport = require('./app/modules/timeReport');
app.registerAdmin(timeReport, {app: app});

```

这里 registerAdmin 可以接收两个或三个参数，如果是三个参数的话，第一个必须是字符串来指定 moduleId。如果是两个参数的话，moduleId 将使用第一个参数，也就是 module 的工厂函数的 moduleId 属性。这里由于我们给 timeReport 定义了 moduleId 属性，因此我们就省略掉了第一个 moduleId 参数了。最后一个参数是配置选项，可以配置监控数据获取是 pull 还是 push 方式，以及获取周期。在我们这个例子中，由于注册时没有传入任何关于 type 和 interval 的配置，将使用默认值，也就是使用拉模式，每隔 5 秒获取一次数据。


## 一些说明

- 在导出一个 module 的时候，一般需要指定一个 moduleId，在这里，我们指定的 moduleId 是 `timeReport`。当然我们如果这里不指定 moduleId 的话，在调用Application.registerAdmin 的时候再指定 moduleId 也是可以的。

- 一个 module 有两个属性很重要，type 和 interval，type 指出的是数据所采用的方式，有两种 pull 和 push。pull 方式是让 master 定时给 monitor 发请求，monitor 给其上报信息。push 的方式则是 monitor 定时上报自己的信息。interval 就是这个信息上报的时间周期了。我们例子中使用的是方式通过 opts 传入，如果 opts 中没有配置的话，默认使用 pull 方式，上报周期为 5 秒，而实际上，我们就是使用了这样的两个参数值，即使用 pull 方式，让 master 主动拉数据，每5秒拉一次。

- 还有一个要注意的地方是 masterHandler 的实现，可能会让人感到迷惑。实际上，由于使用pull 的方式，masterHandler 会在两种情况下被回调，一种是每隔5秒产生的一次拉数据事件，一种是 monitor 向 master 上报信息。这两种情况，可以通过参数 msg 区分。

    - 如果是定时器产生的周期性的拉数据事件导致的回调，此时 msg 参数是 undefined，因此此时只是简单的调用 notifyAll，参数 moduleId 使用来区分到底是哪个监控模块；testMsg 参数在这里仅仅用来示例如何传参,在 monitorHandler 中也仅仅把其打印到 console 上而已，实际应用中，可以用其传递更有意义的参数；

    - 如果是 monitor 在收到 master 的通知后，上报自己的时间信息的话，此时 msg 将会是一个对象，这个时候，master 将这个时间值打印到 console，并缓存其值，当然这个值没什么意义，仅仅是为了示例。因此这段代码通过对msg的判断区分了这两种情况。

    - 实际应用中，也经常使用判断 msg 来区分两种情况的方式。考虑另一种情况，假如使用的不是 pull 方式，而是 push 方式的话，那么 monitor 将会遇到两种情况，与 master 类似，一种是定时器的周期事件，一种是 master 给其发了通知或请求，此时也可以通过判断 msg 进行两种情况的区分，只不过此时将会在 monitorHandler 中进行判断了。关于这种使用 push 方式并在 monitorHandler 中通过判断 msg 的值进行区分两种情况的实现方式，读者可以自行尝试。

- monitorHandler 的实现中，当收到 master 的通知后，取出了 master 传来的参数，这里的参数就是 testMsg，实际应用中可以使用更复杂的更有实际意义的参数。然后通过对参数进行分析，执行相应的逻辑。这里的逻辑很简单，就是获取自己当前的时间，然后通知给 master。

- clientHandler 是当有第三方监控客户端给 master 发请求时，由 master 进行回调的。为了保持简单，我们这里不再对 client 做过多的介绍,在开发指南部分会有详细的介绍。

## 小结

在这部分里，我们使用了 pomelo 提供的监控管理框架完成了 monitor 向 master 上报其本地时间的功能。实际上，通过定制自己的 **admin module** 可以实现上报任何我们需要的上报的数据。比如，在实际应用中，connector 服务器可以向 master 报告登录到其服务器上的用户信息，monitor 可以向 master 上报其进程相关的信息等等。在 pomelo-admin 中还实现了另外几个 admin module，这些 admin module可 以通过对 Application 调用 `app.enable('systemMonitor')` 完成开启，这里不再详述，可以直接阅读相关代码。到此为止，我们基本上就介绍完了pomelo的所有基本功能，下面会有一个简单的[总结还有一些没有涉及到的内容](总结 "总结")。