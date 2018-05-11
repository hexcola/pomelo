pomelo has its own style and conventions, a brief summary is presented as follows:

* Pomelo is a framework, so all the code we write is used to configure the framework and define some callback function which will be invoked by the framework.

* When developing with pomelo, usually, we export a factory function rather than an object while implementing artifact such as component, handler, filter, admin module and remote. It is more flexible to use a factory function than an object because we can do context injected and pass some options to these artifacts when invoking factory function to create it.

* Based on the principle "convention over configuration", we have some specifications on code orgnization. All the server-related code should be placed at "<ProjectDir>/game-server/app/servers/<ServerType>", which includes one or both of handler and remote, they are placed into separated different subdirectory of server directory called handler and remote. Therefore, directory structure is very important in development based on pomelo.

* The naming style of pomelo is similar to Java, In general, use ClassNamesLikeThis, methodNamesLikeThis, CONSTANT_VALUES_LIKE_THIS, variableNamesLikeThis, etc. This style is very common in many programming languages.

* There are also other rules and conventions in pomelo, you can summarize it and make a contribution, all your contributions is welcome.
