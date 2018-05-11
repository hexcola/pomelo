* Are there any performance issues with nodejs comparing to traditional languages like c++ etc?

> Node.js is built on chrome's v8 javascript engine which is fairly fast. The io advantage of node.js is amazing, which is better than any traditional language. The only disadvantage is dealing with compute-intensive problems, which we can solve with a well-designed architecture. In our actual game development, node.js performs better than the traditional languages.

* Do I have to develop using node.js on the server side because of using pomelo framework?

> At present, pomelo does not support multi-language extension. But the language which can be compiled to js 
like coffeescript is ok.  

* What operating system does the pomelo framework support?

> Linux, Windows and Mac os.

* Does pomelo framework work well in projects without javascript on client side?

> Yes, it does. Almost all client languages are supported by pomelo because it is built on socket.io,  which supports clients of almost all languages. You can read reference in [wiki of socket.io](https://github.com/LearnBoost/socket.io/wiki) 

* Is there any difference between `pomelo start` and `cd game-server && node app` to start game server?

> Generally, `pomelo start` is recommended because it records the started server states like daemon and production/development mode, and `pomelo stop` will work 
well to shutdown all the servers. if you use `cd game-server && node app`, there may be port conflicts when restart a new project because the servers are not thoroughly killed.

* How can I deal with the port conflicts caused by a project already running in the backgroud?

> If the servers run in development mode, you can use ‘pomelo kill’ or ‘pomelo kill –force’ to kill the background processes. If in production mode, you should use ‘pomelo stop’ which is more safer.
 
* How to add parameters to the command line for some process?

> Modify configuration file './game-server/config/server.json' and add parameters to target server. For instance, you can add 
parameters to connector server:

```json
 {"connector":[{"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "wsPort":3050, 
"args":"--debug=[port] --trace --prof --gc"}]}
```

* What is the difference between development and production environment?

> By default, the project runs in development environment. If running in production environment,
the project needs start parameter 'production'. And the parameter '--daemon' can make the project run in background, which requires the module 'forever' be installed

* How can you extend servers in production environment?

> If you just want to extend server numbers, you can simply add the server configuration to './game-server/config/server.json'. If you want to divide some business logic to other servers, things will be a little complicated. 

* What can I do if I can not login to the game lordofpomelo in local deployment?

> Please checkout your browser and make sure it supports websocket which can be tested in the website of http://websocketstest.com.
If any port conflicts, please modify the configuration file './game-server/config/server.json'.

* How do I contribute to pomelo?

> Welcome anyone contribute code to pomelo, we will put your name on the contributor list. You can follow us on github, and contribute code or modules to pomelo project.