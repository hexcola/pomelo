* In the last section, we applied dictionary-based route compression on our example application chat to reduce  a lot of communication overhead. Here, we will use protobuf-based encoding/decoding on transmission message to reduce communication overhead, too. Protobuf is a data exchange format suggested by google, more information about protobuf, see [here] (https://code.google.com/p/protobuf/).

* If you use the original protobuf, you may have to follow the next steps: first define a .proto file, and then call protoc to compile it to generate source code  according to the host language, and at last, use the generated code to the applications. This way to use protobuf is relatively heavy, because it involves a static compiling and once the data format has changed, you have to modify the .proto and redo the steps metioned before.

* Pomelo implements a variant of protobuf, it parse the .proto file dynamically at runtime without compling.In pomelo, in order to parse proto file more easily, it uses the json format to represent the message types which has similar syntax with the original proto file. The proto type definitions can be used at client side and server side, the proto file used at server-side should be placed in config/serverProtos.json, while the proto file used at client side should be placed in config/clientProtos.json. If we define a proto type for a kind of message, then it will be encoded/decoded to/from binary data format based on protobuf while transferring. If a certain type of message that does not have a corresponding proto type definition, pomelo will still use the json format.

Using in Chat
============

Now, we will apply protobuf-based encoding/decoding to our example application chat. The code is in the branch `tutorial-protobuf`, use the following command to switch:

    $ git checkout tutorial-protobuf

* First, extract all the message types used in our chat appliction, including server-side's and client-side's, and write them to clientProtos.json and serverProtos.json, as below:

```javascript

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
* Then place these two files into game-server/config;

* Enable protobuf encoding/decoding in our application, configure in app.js:

```javascript

app.configure ('production|development', 'connector', function() {
  app.set ('connectorConfig', {
    connector: pomelo.connectors.hydridconnector,
    heartbeat: 3,
    useDict: true,
    useProtobuf: true // enable useProtobuf
  });
});

app.configure ('production|development', 'gate', function() {
  app.set ('connectorConfig', {
    connector: pomelo.connectors.hybridconnector,
    useDict: true,
    useProtobuf: true // enable useProtobuf
  });
});

``` 

Here is just an example. In fact, for `onAdd` and `onLeave`, the message size is very small and it is a string,  Using protobuf encoding/decoding on them will not reduce the communication overhead observably, but bring the overhead of encoding/decoding, it is unworthy. In practice, we should find a balance between transferring overhead and encoding/decoding overhead according to the actual cases.

For those message types which don't have a proto type definition in .proto file, pomelo will still use the original json format to communicate.

Summary
============

So far, we have implemented a chat application with basic functionalities. We have applied filter mechanism, dictionary-based compression for route, protobuf-based message encoding/decoding on our chat application. In the next sections, we will do some purely "superfluous" work on our chat example to show other features of pomelo, the first one is RPC invocation, and next we will [add an rpc invocation to our example chat application](Rpc-invocation "rpc invocation").
