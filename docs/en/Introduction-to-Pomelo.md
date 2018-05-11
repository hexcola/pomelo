# Introduction to Pomelo


### What is Pomelo?
Pomelo is a fast, scalable, distributed game server framework for [Node.js](http://nodejs.org).
It provides the basic development framework and a lot of related components, including libraries and tools. 
Pomelo is also suitable for realtime web application, its distributed architecture makes Pomelo scale better than other realtime web frameworks.

The following is the composition of Pomelo:

<center>
 ![pomeloFramework](http://pomelo.netease.com/resource/documentImage/pomeloFramework.png)
</center>

Pomelo includes the following parts:
* Framework is the core of Pomelo, it is a scalable, distributed game server framework, and really easy to use.
* Libraries, we provide a lot of libraries for game development, including AI, path finding, AOI(area of interested) etc.
* Tools, we provide a lot of tools, including admin console, command line tool, stress testing tool.

### Features
#### Fast, scalable

* Distributed (multi-process) architecture
* Flexible server extension
* Full performance optimization and test

#### Easy

* Simple API: request, response, broadcast, etc.
* Lightweight: high development efficiency based on node.js
* Convention over configruation: almost zero config

#### Powerful

* Many libraries and tools
* Good reference materials: full docs, and [an open-source MMO RPG demo](https://github.com/NetEase/pomelo/wiki/Introduction-to--Lord-of-Pomelo)

### Why should you use Pomelo?
Fast, scalable, realtime game server development is not an easy job. A good container or framework can reduce the complexity.
Unfortunately, not like web, the game server framework solution is quite rare, especially open source. Pomelo will fill this blank, providing a full solution for building game server framework.
The following are the advantages:
* The architecture is scalable. It uses multi-process, single thread runtime architecture, which has been proved in industry and  especially suitable for Node.js thread model.
* Easy to use, the development model is quite similiar to web, using convention over configuration, almost zero config. The API is also easy to use.
* The framework is extensible. Based on Node.js micro module principle, the core of Pomelo is small. All the components, libraries and tools are individual npm modules, anyone can create their own module to extend the framework.
* The reference is quite complete, we have complete documents. In addition to documents, we also provide a full open source MMO demo(HTML5 client), which is a far more better reference than any books.


### How to develop with Pomelo?
With the following references, we can quickly familiar the Pomelo development process:
* [The architecture overview of pomelo](https://github.com/NetEase/pomelo/wiki/Architecture-overview-of-pomelo)
* [Quick start guide](https://github.com/NetEase/pomelo/wiki/Quick-start-guide)
* [Tutorial](https://github.com/NetEase/pomelo/wiki/Tutorial)
* [FAQ](https://github.com/NetEase/pomelo/wiki/FAQ)

You can also learn from our MMO demo:
* [An introduction to demo --- Lord of Pomelo](https://github.com/NetEase/pomelo/wiki/Introduction-to--Lord-of-Pomelo)
