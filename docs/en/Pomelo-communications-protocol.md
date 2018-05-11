##Pomelo protocol

###Background

Pomelo uses json as the communication protocol between client and server in the 0.2 versions. The protocol based on json is flexible and easy to modify. But it also introduces a lot of redundant data. Pomelo 0.3 replaces the json string protocol with a new binary protocol and makes great efforts on content compression to improve the efficiency of bandwidth.

The new Pomelo protocol contains to layer encoding: `package` and `message`. The message layer provides the route string compression and protobuf compression. And the encoding result from message layer would be passed to the package layer. 

The package layer provides a series mechanism of handshake, heartbeat and data transmission based on binary stream. The encoding result from package layer is free to transmitted on TCP or websocket.

Both of the message layer and package layer could be replaced independently since neither of them relies on the other one directly.

The layers of Pomelo protocol is as below:

<center>
![Pomelo Protocol](http://pomelo.netease.com/resource/documentImage/protocol/Data transport.png)
</center>

###Pomelo Package

There are two kinds of package run on the package layer: control package and data package. 

Control package is used to the control operaion, such as handshake, heartbeat and disconnect message from server.

While the data package is mainly used to transmit data between cilent and server.

####Package format

Package is composed by two parts: header and body.

Header contains the descriptions of the package, such as the package type and the length of package body.

Body contains the binary payload of the package.

The format of package:

<center>
![pomelo package](http://pomelo.netease.com/resource/documentImage/pomelo-package.png)
</center>

+ type - package type, one byte
	* 0x01: handshake request from client to server and handshake response from server to client.
	* 0x02: handshake ack from client to server.
	* 0x03: heartbeat package.
	* 0x04: data package.
	* 0x05: disconnect message from server.
+ length - length of body in byte, 3 bytes big-endian integer.
+ body - binary payload.

The mainly process flow of each package type as below:

####Handshake

Handshake phase provides a chance to exchange initialization data between client and server after the connection established on the application level.

The handshake data splited into two parts: system and user. The system data is used by Pomelo framework and the user data is customized by the upper level application.

The handshake data is encoded as a utf-8 json string without any compression and transmitted by the body of the handshake package.

A handshake request is as following:

```
{
  'sys': {
    'version': '1.1.1',
    'type': 'js-websocket'
  }, 
  'user': {
  	// any customized request data
  }
}
```

+ sys.version - client version. Each version client SDK should assign a constant version number and should upload this version number to the server during the handshake phase. And the server could use this information to figure out whether the client is suitable to communicate with.
+ sys.type - client type, such as c, android and ios. Used with `sys.version` together to check the client.

A handshake response is as following:

```
{
  'code': 200, 			// result code
  'sys': {
    'heartbeat': 3, 	// heartbeat interval in second
    'dict': {}, 		// route dictionary
    'protos': {}		// protobuf definition data
  }, 
  'user': {
  	// any customized response data
  }
}
```

+ code - response code of handshake, such as 200 for ok, 500 for handshake fails and 501 for invalid client type and version.
+ sys.heartbeat - optional heartbeat interval in second. Empty for no heartbeat.
+ dict - optional route dictionary that used for route string compression. Empty for no route string compression.
+ protobuf - optional protobuf definitions. Empty for no protobuf compression.
+ user - optional customized handshake data. Empty for no customized handshake data.

The process flow of handshake as below:

<center>
![handshake](http://pomelo.netease.com/resource/documentImage/pomelo-handshake.png)
</center>

####Heartbeat

The length of heartbeat package is 0 and the body is empty.

The process flow of heartbeat as below:

<center>
![heartbeat](http://pomelo.netease.com/resource/documentImage/pomelo-heartbeat.png)
</center>

Congiure the heartbeat interval in server. And server would fire the first heartbeat after handshake. And then both of server and client delays a heartbeat interval and then sends a heartbeat request to the other after receiving a heartbeat request.

The heartbeat timeout is double of the heartbeat interval. Server would not break the connection when it detectes a heartbeat timeout. And the client can make it desision whether to break the connection when it detectes a heartbeat timeout. 

####Data

Data package is used to transmit binary data between client and server. The payload of data package could be any binary data from the upper layer. The package layer would not do any modifcation of the payload.

####Disconnect by server

When server wants to break a client connection, such as kicking a player offline, it would send a control package, known as kick package, and then breaks the connection.

The client can figure out that the connection is broken by the server by this package.

###Pomelo Message
The Pomelo Message protocol is used to build the message head, which includes message route and message type. Different type of messages have different head.

#### The format of message head
Message head is composed by three parts: flag, message id, route, as in the image below. 

<center>
![Message Head](http://pomelo.netease.com/resource/documentImage/protocol/MessageHead.png)
</center>

Flag is used to identify the message type and other information. Message id is a unique id for the message, used in the request/response mode. Message route is used to find idnetify the handler of the message, used at request/response mode and server publish mode.

As in the above picture, the flag field is necessary for message head, it will occupy a byte, it decide the other fields of the head. Message id and message route are exchangeable, the length and content are decided by message flag.
##### Flag field
Flag field always at the first byte of message head, it's content is as follow:

<center>
![flag](http://pomelo.netease.com/resource/documentImage/protocol/MessageFlag.png)
</center>

In pomelo 0.3, we only used 4 bits of the flag field, and the 4 bits can be divided into two parts: message type field for 3 bits and route compression flag for 1 bit.

* Message type filed is used to identify the message type,it can represent 8 kinds of message with value from 0 to 7, now we only use 4 kinds of message with the value from 0~3. The message type and there meaning will be explanation later.
* Route compression flag is used to identify if the route field in the message head is compressed, it will affect the length of the route field.

#####Message type filed
The value of the message type filed represent the kinds of different messages, as in the follow form:

<center>
![Message Type](http://pomelo.netease.com/resource/documentImage/protocol/MessageType.png)
</center>

And different message type may have different message head, as in the follow picture :

<center>
![Message Head Content](http://pomelo.netease.com/resource/documentImage/protocol/MessageHeadContent.png)
</center>

#####Route field
We use the route flag in the flag field to identify if the route is compressed, where 1 means a compressed route and 1 means no compressed. The contents of different route are as follow :  

<center>
![Message Type](http://pomelo.netease.com/resource/documentImage/protocol/MessageRoute.png)
</center>

As we can see in the above picture:
* For a compressed route, the route is a uInt16.
* For a original route, we use an uInt8 to identify it's length(0~255), which followed by route content encoded with utf8.

For more information of route compression, see at [Pomelo data compression protocol](https://github.com/NetEase/pomelo/wiki/Pomelo-data-compression-protocol).