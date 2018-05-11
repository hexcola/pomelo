In practice, network bandwidth is a worthwhile consideration. Especially for mobile clients, the network resource is often not very rich, in order to save network resources, it is often needed to increase the effective payload ratio.

Using the chat application as an example, when a user send a chat message, the route information is required, as shown below:

```javascript

pomelo.request('chat.chatHandler.send',
  //...
);

```
The routing information indicates that the request should be handled by send method of chatHandler on chat server. When server pushing messages to the client, route also should be specified to indicate a handler. In the chat example, there are onAdd, onLeave and other routes. Considering if a chat message is very short such as just a letter, but when being sent, it should be added a complete routing information, this would result in a very low effective payload ratio and wasting network resource. The direct idea to solve this problem is to shorten the routing information. On the server side, routing information is fixed while the server is determined. On the client side, although you can use a very short name for route, but it may be unreadable. 

To address this situation, pomelo provides the dictionary-based route compression.

* For the server side, pomelo scans all route information;
* For the client side, the developer needs list the route in file config/dictionary.json.

Then, pomelo would get all the routes of client side and server side and then maps each route to a small integer. Currently pomelo route compression only supports hydridconnector. In implementation of hydridconnector, if you enable route compression, then the client and server will synchronize the dictionary in handshake phase while establishing a connection, so that the small integer can be used to replace the route later, and it can reduce the transmission cost.


Using in Chat
===========

Now, we enable the route compression for our chat example. The sample code is specific in the branch `tutorial-dict`, use the following command to switch branch:

     $ git checkout tutorial-dict

First, extract all the route from client side and list it in config/dictionary.json:

```javascript

// dictionary.json
[
  "onChat",
  "onAdd",
  "onLeave"
]

```

Then enable route compression by configuring connector options:

```javascript

app.configure('production|development', 'connector', function() {
  app.set ('connectorConfig', {
    connector: pomelo.connectors.hydridconnector,
    heartbeat: 3,
    useDict: true // enable dict
  });
});

app.configure('production|development', 'gate', function() {
  app.set ('connectorConfig', {
    connector: pomelo.connectors.hydridconnector,
    useDict: true // enable dict
  });
});

```

* Well, now we have enabled routing compression for our chat example, and all the route will be replaced to a small integer while transferring.
*  If some routes at client side is not listed in the dictionary.json, then pomelo would still use the original route without compression .

Summary
=========

So far, The format of transmission message between client and server is json. Indeed, while json is very convenient, but it also brought some redundant information, which can be ommited to reduce transmission cost. That means directly sending binary data will be better to reduce overhead. In the next section we will apply [protobuf encoding/decoding] (Protobuf-compression "Protobuf compression") on our chat application in order make it better.
