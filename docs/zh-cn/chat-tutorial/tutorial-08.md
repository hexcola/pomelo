---
title:  给 pomelo 加个组件
---

<!-- TOC -->

- [chat 中应用](#chat-中应用)
- [一些说明](#一些说明)
- [小结](#小结)

<!-- /TOC -->

pomelo 的核心是由一系列松耦合的 component 组成，同时我们也可以实现我们自己的component 来完成一些自己定制的功能。对于我们的 chat 应用，我们尝试给其增加一个 component，目的是展示如何增加一个 component，以及 component 的生命周期管理，而不会特别关注这个 component 的实际功能。我们现在就给其增加一个 component `HelloWorld`，这个 component 仅仅在 master 服务器上加载运行,在 master 服务器的话，它将每隔一段时间在console 上打印出来一个 `HelloWorld`，具体的时间间隔由 opts 配置来指定。

## chat 中应用

具体的代码在分支 `tutorial-component` 上，使用如下命令切换分支：

```bash
$ git checkout tutorial-component
```

- 首先，在 `app` 下建立 `components/HelloWorld.js` 文件, 大致代码如下：

```js

// components/HelloWorld.js
module.exports = function(app, opts) {
  return new HelloWorld(app, opts);
};

var DEFAULT_INTERVAL = 3000;

var HelloWorld = function(app, opts) {
  this.app = app;
  this.interval = opts.interval || DEFAULT_INTERVAL;
  this.timerId = null;
};

HelloWorld.name = '__HelloWorld__';

HelloWorld.prototype.start = function(cb) {
  console.log('Hello World Start');
  var self = this;
  this.timerId = setInterval(function() {
    console.log(self.app.getServerId() + ": Hello World!");
    }, this.interval);
  process.nextTick(cb);
}

HelloWorld.prototype.afterStart = function (cb) {
  console.log('Hello World afterStart');
  process.nextTick(cb);
}

HelloWorld.prototype.stop = function(force, cb) {
  console.log('Hello World stop');
  clearInterval(this.timerId);
  process.nextTick(cb);
}

```

我们看到每一个 component 一般都要定义 `start`，`afterStart`，`stop` 这些 hook 函数，供 pomelo 管理其生命周期时进行调用。对于 component 的启动，pomelo 总是先调用其加载的每一个 component 提供的 start 函数，当全部调用完后，才会去调用其加载的每一个component 的 afterStart 方法，这里总是按顺序调用的。在 afterStart 中，一些需要全局就绪的工作可以放在这里完成，因为调用 afterStart 的时候，所有 component 的 start 已经调用完毕。stop 用于程序结束时对 component 进行清理时使用。虽然我们这个例子非常简单，但是足以说明如何在 pomelo 中定制自己的 component，并使用。我们在 HelloWorld 的 start 里面启用了一个定时器，每隔一段时间就向 console 打印一个 HelloWorld。然后在 stop 里关闭它。

- 然后，我们让 master 服务器来加载我们的这个 component，具体代码如下：

```js
// app.js
var helloWorld = require('./app/components/HelloWorld');

app.configure('production|development', 'master', function() {
  app.load(helloWorld, {interval: 5000});
});

```

好了， 我们通过实现一个简单的 component，明白了如何实现自己的定制 component，当然这个 component 很简单，也没有啥实际意义，但是它是一个完整的 component，有完整的生命周期管理，我们通过这个例子已经了解到了 component 的创建以及加载。

## 一些说明

- 这里定义的 HelloWorld component 中，往外导出的是一个工厂函数，而不是一个对象。当app 加载 component 时，如果是一个工厂函数，那么 app 会将自己作为上下文信息以及后面的opts 作为参数传给这个函数，使用这个函数的返回值作为 component 对象。同样，也可以直接给 `module.exports` 赋予一个对象，那样的话，就可以直接使用而不用调用工厂函数，不过这样的话丧失了使用 app 和具体配置参数， 不够灵活，因此，使用工厂方法的方式是一个好选择。

- 对于 start 和 afterStart 的执行，pomelo 总是会先按顺序执行完所有 component 的start 后，才会按顺序执行所有 component 的 afterStart，因此可以在 afterStart 里执行一些需要其他 component 执行了 start 后才可以执行的逻辑。

- 实际上，pomelo 应用的整个运行过程可以认为是管理其 component 的生命周期过程，pomelo的所有功能都是通过其内建的component来实现的。用户可以轻松地定制自己的 component，然后将其 load 进去，这样就很轻松地实现了对 pomelo 的扩展。

## 小结

在这部分，我们给聊天应用 “画蛇添足” 般地添加了一个 HelloWorld component，使得可以更好地理解如何定制自己的 component，并加载它。pomelo 的框架非常灵活，有很好的可扩展性，从我们的例子中，我们可以看出，可以很容易地对 pomelo 进行扩展。pomelo 不仅可以扩展component，pomelo 还提供了一个灵活的可扩展的服务器监控管理框架，[下一步](增加admin-module "增加admin-module")，我们将给我们的聊天应用增加一个监控模块，让应用服务器自动向 master 上报自己的本地时间，以此来示例如何在 pomelo 中定制自己的监控管理模块。