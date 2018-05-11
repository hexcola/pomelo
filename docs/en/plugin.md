## Compatibility

Pomelo plugins were introduced in version `0.6`.

## Plugin Structure

Pomelo plugins are divided into two sections: **components** (required) and **events** (optional):

![plugin](http://pomelo.netease.com/resource/documentImage/plugin.png)

In Pomelo, a component is a reusable factory where each instance provides a service. For more in-depth information on Pomelo component architecture, see [What is a component](https://github.com/NetEase/pomelo/wiki/Pomelo-Framework-Reference).

Each plugin can have several components, and for each component developers can implement different life-cycle callbacks (e.g. `start`, `afterStart`, `stop`, etc.).

Each plugin dispatches the following events:

- `'add_servers'`： event parameter is an Array of server information.
- `'remove_servers'`: event parameter is an Array of server information.
- `'replace_servers'`: internal server disconnect and reconnect event. Parameter is an Array of server information which have not disconnected.
- `'bind_session'`: session has been bound. Parameter is the session object.
- `'close_session'`: close session event (including connection error). Parameter is the session object.

## Using a Plugin

Most plugin configuration will occur in your `app.js` file.

####`app.use(plugin, options);`

- `'plugin'`: plugin definition
- `'options'`: plugin options

Example:

    var statusPlugin = require('pomelo-status-plugin');
    ...
    app.use(statusPlugin, {
      status: {
        host: '127.0.0.1',
        port: 6379
      }
    });
    ...

##Building a Plugin

First, create a plugin project with directory structure like so:

![plugin-dir](http://pomelo.netease.com/resource/documentImage/plugin-dir.png)

Second, you need to configure the components and events in `index.js`:

    module.exports = {
      components: __dirname + '/lib/components/',
      events: __dirname + '/lib/events/'
    };

Finally, you can add your components and events. For components, you need to export its constructor, the example code is as follow:

    module.exports = function(app, opts) {
      return new Component(app, opts);
    };

    var Component = function(app, opts) {
      //do construction
    };

    Component.prototype.start = function(cb) {
      // do something application start
    };

    Component.prototype.afterStart = function(cb) {
      // do something after application started
    };

    Component.prototype.stop = function(cb) {
      // do something on application stop
    };

Events are similar:

    module.exports = function(app) {
      return new Event(app, opts);
    };

    var Event = function(app) {
      //do construction
    };

    Event.prototype.add_servers = function(servers) {
      //do something when application add servers
    };

    Event.prototype.remove_servers = function(ids) {
      //do something when application remove servers
    };

    Event.prototype.replace_servers = function(servers) {
      //do something when server reconnected
    };

    Event.prototype.bind_session = function(session) {
      //do something when session binded
    };

    Event.prototype.close_session = function(session) {
      //do something when session closed
    };