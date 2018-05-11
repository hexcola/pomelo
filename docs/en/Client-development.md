In order to facilitate client development, pomelo provides client SDKs for some platforms, this document mainly introduce the JavaScript SDK for Web platform and libpomelo which is written in C language and can be used in many platforms.

Actually, if the client uses the same wire-protocol as the server, then they can communicate with each other. Pomelo provides two builtin connector sioconnector and hybridconnector, they use different wire-protocols, sioconnector is based on socket.io, while hybridconnector can handle connection for tcp and websocket. If developers customize their own wire-protocol and connector, then they should implement a corresponding client too. 

JavaScript SDK for Web 
=======================

Websocket is already supported for HTML5, so if a browser supports websocket, then it can directly connect to server via hybridconnector. But for an older browser which does not support websocket, socket.io based communication is a better choice. Therefore, there are two SDK provided for Web client: one for browsers which support websocket and another for browsers which do not support websocket, these two SDK are [pomelo-jsclient-socket.io] (https://github.com/pomelonode/pomelo-jsclient-socket.io) for socket.io and [pomelo-jsclient-websocket](https://github.com/pomelonode/pomelo-jsclient- websocket) for websocket.

#### Socket.io-based Client SDK

For socket.io-based client SDK, it depends on [socket.io-client] (https://github.com/learnboost/socket.io-client/), As there are some bugs when managing this dependency by [component] (https://github.com/component/component/), so it is required to use js file directly, the js file is [socket.io-client.js](https://github.com/LearnBoost/socket.io-client/ blob/master/socket.io-client.js). 

The js file for pomelo-jsclient-socket.io is [pomelo-client.js](https://github.com/pomelonode/pomelo-jsclient-socket.io/blob/master/lib/pomelo-client.js). It is required to reference these 2 js file before pomelo is able to work.

#### Websocket Client SDK 

For websocket client SDK, it uses the [component] (https://github.com/component/component/) to manage its dependencies, so it is only required to configure a component.json file, in which configuring the dependencies, and then execute: 

    $ component install
    $ component build

component will automatically find all the configured dependencies and package them into a single js file. Developers only need to reference the compiled build.js, and then pomelo is able to work. Here is an example [chatofpomelo-websocket] (https://github.com/NetEase/chatofpomelo-websocket/tree/master/web-server/public/js/lib), in which it uses component to manage the js dependencies at front end.


<a name="clientAPI"/>
#### Javascript API 

Pomelo provides the unified API for both socket.io and websocket, the following is a brief introduction about the API.

* pomelo.init(params, cb)

This is often the first call at client side, the server address and port should be indicated in params, and cb will be called back after successfully connecting;

* pomelo.request(route, msg, cb)

It is called if client wants to send a request, route indicates service location formatted "<ServerType>.<HandlerName>.<MethodName>", and msg is content of the request, cb will be called back after receiving a response;

* pomelo.notify(route, msg)

It is called if client wants to send a notify, it does not need a response from server side, so it has no cb parameter, the other parameters have the same meaning as pomelo.request;

* pomelo.on(route, cb)

This is a method inheriting from EventEmmiter, it is used to install handler to respond the pushing message pushed by server-side. route is be user-defined formatted "onXXX", cb is the handler;

* pomelo.disconnect()

It would be called when client wants to break the connection.

libpomelo
===========

libpomelo is client SDK for [pomelo](https://github.com/NetEase/pomelo) written in C language that supports higher version than 0.3 of pomelo. 

### Dependencies
* [libuv] (https://github.com/joyent/libuv), a cross-platform library that can be used for network I/O and thread managing.
* [jansson] (https://github.com/akheron/jansson), a json parsing library written in C language. 

### How to Use 

#### Creating a Client Instance

```
// create a client instance.
pc_client_t *client = pc_client_new();
```

#### Adding Event Listeners

```
// Add some event callback.
pc_add_listener(client, "onHey", on_hey);
pc_add_listener(client, PC_EVENT_DISCONNECT, on_close);
```

#### Listener Definition

```
// disconnect event callback.
void on_close(pc_client_t *client, const char *event, void *data) {
  printf ("client closed: %d. \n", client->state);
}
```

#### Connecting to Server

```
struct sockaddr_in address;

memset(&address, 0, sizeof (struct sockaddr_in));
address.sin_family = AF_INET;
address.sin_port = htons(port);
address.sin_addr.s_addr = inet_addr(ip);

// try to connect to server.
if(pc_client_connect(client, &address)) {
  printf ("Fail to connect server. \n");
  pc_client_destroy (client);
  return 1;
}
```

#### Sending a Notify 
```
// notified callback
void on_notified (pc_notify_t * req, int status) {
  if (status == -1) {
    printf ("Fail to send notify to server. \ n");
  } else {
    printf ("Notify finished. \ n");
  }

  // release resources
  json_t *msg = req-> msg;
  json_decref (msg);
  pc_notify_destroy (req);
}

// send a notify
void do_notify(pc_client_t * client) {
  // compose notify.
  const char *route = "connector.helloHandler.hello";
  json_t *msg = json_object();
  json_t *json_str = json_string ("hello");
  json_object_set (msg, "msg", json_str);
  // decref json string
  json_decref(json_str);

  pc_notify_t *notify = pc_notify_new();
  pc_notify(client, notify, route, msg, on_notified);
}
```

#### Sending a Requst 

```
// request callback
void on_request_cb (pc_request_t * req, int status, json_t * resp) {
  if(status == -1) {
    printf ("Fail to send request to server. \n");
  } else if(status == 0) {
    char *json_str = json_dumps(resp, 0);
    if (json_str != NULL) {
      printf("server response:% s \n", json_str);
      free(json_str);
    }
  }

  // release relative resource with pc_request_t
  json_t * msg = req-> msg;
  pc_client_t * client = req-> client;
  json_decref (msg);
  pc_request_destroy (req);

  // stop client
  pc_client_stop (client);
}

// send a request
void do_request(pc_client_t * client) {
  // compose request
  const char *route = "connector.helloHandler.hi";
  json_t *msg = json_object();
  json_t *str = json_string("hi ~");
  json_object_set(msg, "msg", str);
  // decref for json object
  json_decref(str);

  pc_request_t *request = pc_request_new();
  pc_request(client, request, route, msg, on_request_cb);
}
```

### API 
* Creating an instance of pomelo client:
```
pc_client_t *pc_client_new();
```

* Stopping client's connection:

The function is suitable to be called in child thread of libuv, and then main thread would wait for child thread's exit by calling pc_client_join.

```
void pc_client_stop(pc_client_t *client);
```

* Destroying client's connection:
```
void pc_client_destroy(pc_client_t *client);
```

* Waiting for child thread's exit for main thread:

```
int pc_client_join(pc_client_t *client);
```

* Creating an request instance:
```
pc_request_t *pc_request_new();
```

* Destroying an request instance:
```
void pc_request_destroy(pc_request_t *req);
```

* Connecting to server, it would create child thread to handle network I/O:
```
int pc_client_connect(pc_client_t *client, struct sockaddr_in *addr);
```

* Destroying an instance of type pc_connect_t: 
```
void pc_connect_req_destroy(pc_connect_t *conn_req);
```

* Sending a request:
```
int pc_request(pc_client_t *client, pc_request_t *req, const char *route,
               json_t *msg, pc_request_cb cb);
```

* Creating a notify instance:
```
pc_notify_t *pc_notify_new();
```

* Destroying a notify instance:
```
void pc_notify_destroy(pc_notify_t *req);
```

* Sending a notify:
```
int pc_notify(pc_client_t *client, pc_notify_t *req, const char *route,
              json_t *msg, pc_notify_cb cb);
```

* Adding an event listener:
```
int pc_add_listener(pc_client_t *client, const char *event, pc_event_cb event_cb);
```

* Removing an event listener:
```
void pc_remove_listener(pc_client_t *client, const char *event, pc_event_cb event_cb);
```

* Emitting an event:
```
void pc_emit_event(pc_client_t *client, const char *event, void *data);
```

### Compiling libpomelo

#### Prerequisites

[gyp](http://code.google.com/p/gyp/source/checkout) is required.

#### Mac 

```
./pomelo_gyp
xcodebuild -project pomelo.xcodeproj
```

#### IOS 
```
./pomelo_gyp -DTO=ios
./build_ios
```

#### IOS Simulator
```
./pomelo_gyp -DTO=ios
./build_iossim
```

#### Linux 
```
./pomelo_gyp
make
```

#### Windows 

First, change current directory to libpomelo project directory, and then start the [git bash](https://help.github.com/articles/set-up-git # platform-windows), execute the following commands: 

```
mkdir -p build
git clone https://github.com/martine/gyp.git build/gyp
```
Then, run cmd and change current directory to libpomelo directory, then execute the following commands to generate .sln file which can be used by Visual Studio.
```
build\gyp\gyp.bat -depth=. pomelo.gyp -Dlibrary=static_library -DTO=pc
```

#### Android 

If your development platform is windows, there are several prerequisites:
* Install [Cygwin](http://www.cygwin.com/) with make (select make package from the list during the install).
* eclipse with android adt, the lastest android sdk includes a fully configured eclipse.

Building steps:

**I**. Create a new android project, for example test project is shown as follows:
![test project](http://ww3.sinaimg.cn/large/6a98ae6cgw1e3ym2kkdofj207x0f0q3t.jpg)

**II**. Create a new directory named jni and place it in root directory of the test project, and new an file Android.mk in this jni directory. Write the following to Android.mk:

      LOCAL_PATH := $(call my-dir)

      include $(CLEAR_VARS)

      LOCAL_MODULE := game_shared

      LOCAL_MODULE_FILENAME := libgame

      LOCAL_WHOLE_STATIC_LIBRARIES := pomelo_static
       
      include $(BUILD_SHARED_LIBRARY)

      LOCAL_CFLAGS := -D__ANDROID__

      $(call import-module, libpomelo)

**III**. Create a new directory named pomelo in root directory of the project, and then pull the latest libpomelo code from github as follows:  

<center>
![enter image description here](http://ww2.sinaimg.cn/large/6a98ae6cgw1e3ymh7vavqj208h0gtq3v.jpg)
</center>

IV. Then open a terminal (cygwin for windows) and execute the following command in the root directory of the project to complete the compling:

```
ndk-build NDK_MODULE_PATH=<AbstractPathOfTheProject>/pomelo/
```

![enter image description here] (http://ww3.sinaimg.cn/large/6a98ae6cgw1e3ymfi9wk8j20lt0e9787.jpg)

**V**. If you want to develop applications with cocos2d-x, then just put libpomelo in directory <AbsolutePathOfCocos2dx>/cocos2dx/platform/third_party/android/prebuilt and execute ./Build_native.sh.

You can refer to [cocos2d-x android](https://github.com/cocos2d/cocos2d-x/tree/master/samples/Cpp/TestCpp/proj.android) for more detail.
