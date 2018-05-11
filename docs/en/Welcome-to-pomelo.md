Pomelo is an easy to use, fast, scalable, distributed game server framework for Node.js. It provides a core network architecture and a series of tools and libraries that can help developers eliminate boring duplicate work for common underlying logic. The goal of Pomelo is to improve development efficiency by eliminating the need to spend time on repetitious network related programming.

While developed originally for games, Pomelo is not game specific and could be used for any real-time web application. The scalability and flexibility of Pomelo makes it usable as a general-purpose distributed real-time application development framework.

In addition, Pomelo supports several major client SDKs, making communication with the game server(s) a breeze to set up for varying platforms, including:

- Javascript: automatically generated when running the core `pomelo init` setup
- [.Net](https://github.com/NetEase/pomelo-unityclient-socket)
- [Android](https://github.com/NetEase/pomelo-androidclient)
- [iOS](https://github.com/NetEase/pomelo-ioschat)

While several of these are under construction, they can certainly be used for example code. We are also looking for contributors.

Composition of Pomelo 
=====================

* #### Framework
The network framework is the core of pomelo.

* #### Libraries
Pomelo provides a lot of libraries. These include game-specific AI, AOI, path-finding, etc. as well as general functionalities like timing task execution, data synchronization, etc.

* #### Tools
Pomelo provides a lot of tools, including a server management & control tool, command-line tools (`pomelo list`, `pomelo kill`, `pomelo stop`, for example), a stress testing tool, etc.

* #### Client SDKs
Pomelo provides client SDKs for all major platforms, including JavaScript, C, C#, Android, iOS, and Unity3D. As the communication protocol of pomelo is open and customizable, developers can easily customize their own communication protocol, so pomelo can be extended to support any client platform.

* #### Demo
Pomelo offers several demos to help with developer ramp-up:
- [Chat of Pomelo](https://github.com/NetEase/chatofpomelo): a chat demo that can run on all the major platforms and includes channels, multiple connector servers, etc.
- [Treasures](https://github.com/NetEase/treasures): a simple game demo and a HTML5 client.
- [Lord of Pomelo](http://pomelo.netease.com/lordofpomelo/ "Lordofpomelo"): a full HTML5-based MMO game demo  ([source code](https://github.com/NetEase/lordofpomelo "Lordofpomelo source")) .

Why Pomelo?
===========

Developing a high concurrency real-time game server is a complex task. Up until recently, there were few  suitable open source solutions in the field of game server development. Pomelo aims to fill that need by providing:

* #### High Scalability
Pomelo introduces a single-threaded multiprocess architecture. Each server is its own node-based process. It is easy to add or remove servers to an existing cluster by modifying the configuration file, without requiring any changes to the source code of the application.

* #### Easy to Use
Pomelo is based on Node.js and was developed similar to a web application (think Express). Similar to Ruby on Rails, Pomelo follows the "convention over configuration" principle and requires virtually no configuration out of the box to get a basic application running.

* #### Loosely Coupled and Highly Extensible
Following the micro module principle of Node.js, Pomelo itself has a small core. All the components, libraries, and tools are provided as loosely coupled NPM modules that extend the core framework. We encourage third parties to develop their own Pomelo extension modules.

* #### Documentation
All major functionality is documented here in this Wiki. We welcome any suggestions or questions so we can improve your experience with the framework.

* #### Complete MMO Demo
Pomelo includes a complete open source MMO game demo called [Lord of Pomelo] (http://pomelo.netease.com/lordofpomelo/ "Lord of pomelo online") ([Source Code](https://github.com/NetEase/lordofpomelo "Lordofpomelo source " )), which consists of more than 10k lines of JavaScript.

Pomelo Purpose
==============

Pomelo was designed for server-side applications like real-time games, social games, mobile games, etc of all sizes.

Pomelo is not recommended for use when developing large-scale MMORPG game servers, especially large-scale 3D games. For applications like that, the commercial engine such as [Bigworld](http://bigworldtech.com/en/) may be a better choice.

Well, let's [Install pomelo](Installation "pomelo Installation") and try it .