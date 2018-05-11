Pomelo is composed by a series of loosely-coupled component, and we can also implement our own components to support our specific functionality. For our example application chat, we try to add an additional component to it, aiming to demonstrate how to add a component to  pomelo without concerning about the actual functionality of this component. Well, we will add a component `HelloWorld` to pomelo, which will periodically print "HelloWord" to console, and make this component be loaded only on the master server.

Using in Chat 
=============

The code is in branch `tutorial-component`, use the following command to switch:

    $ git checkout tutorial-component

* First, create file component/HelloWorld.js in the directory game-server/app, the content of HelloWorld.js is shown below:

```javascript

// components/HelloWorld.js
module.exports = function(app, opts) {
  return new HelloWorld(app, opts);
};

var DEFAULT_INTERVAL = 3000; // print cycle

var HelloWorld = function(app, opts) {
  this.app = app;
  this.interval = opts.interval | DEFAULT_INTERVAL;
  this.timerId = null;
};

HelloWorld.name = '__HelloWorld__';

HelloWorld.prototype.start = function(cb) {
  console.log('Hello World Start');
  var self = this;
  this.timerId = setInterval(function() {
    console.log (self.app.getServerId() + ": Hello World!");
  }, this.interval);
  process.nextTick (cb);
}

HelloWorld.prototype.afterStart = function(cb) {
  console.log ('Hello World afterStart');
  process.nextTick (cb);
}

HelloWorld.prototype.stop = function(force, cb) {
  cosole.log ('Hello World stop');
  clearInterval (this.timerId);
  process.nextTick (cb);
}

```

There are several callbacks should be defined for a component, start, afterStart and stop, which are used to manage the component's lifecycle. When application launching, pomelo will call the callback `start` for all its loaded components serially, and then `afterStart`. In general, it will do some initialization for the component in the callback `start`, and work which depends the operations in other components' callback `start` can be done in the callback `afterStart`. The callback `stop` would be called while stopping the application, and it will do some cleaning up. 

Although our example is very simple, it would demonstrate how to customize a component and load it. In `HelloWorld` component, it starts a timer to print `HelloWorld` to console periodically, and clean the timer in its callback `stop`.

* Then, make master server load this component, the code is as follows:

```javascript
// app.js
var helloWorld = require ('./app/components/HelloWorld');

app.configure ('production|development', 'master', function() {
  app.load (helloWorld, {interval: 5000});
});

```
Well, we have implemented a simple component and understand how to implement a customized component. Nevertheless, this component is so simple that it has no practical significance, but it is a complete component with full life-cycle .

Some notes
==========

* Here, when defining HelloWorld component, what to be exported is a factory function rather than an object. When the application loads a component, if it is found the exported component is a factory function, then application create the component instance using the factory function with passing itself and options as the arguments for that factory function. Similarly, it can export an component object directly, if so, application will use the object directly. It is a better way to export a factory function when defining a component because it can use specific options and application context when constructing the component.

* In fact, the running process of pomelo application can be considered to manage the lifecycle of all its loaded components. All functionalities of pomelo is provided by its builtin components, and developers can easily customize their own component, and then load into pomelo application. It is easy to extend pomelo framework via customizing a component.

Summary
===========

In this section, we have implemented a "superfluous" component `HelloWorld` to demonstrate how to customize a component and load it. Pomelo framework is very flexible and easy to extend. Pomelo not only provides customizable component, pomelo also provides a flexible and extensible server administration framework. [Next] (adding-an-admin-module "add admin-module"), we will add a admin module to pomelo's administration framework to let application servers automatically report their own local time to master, it is an example of how to customize an admin module for pomelo administration framework. 
