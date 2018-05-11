Warm up
====================

* This tutorial is suitable for beginners, if you have some development experience in pomelo, please skip this tutorial. You can read the developer guide, there will be some topics discussed in detail.

* Since pomelo is based on node.js and use javascript to develop, so we hope you have some familiarity with node.js and javascript before reading this tutorial.

* The tutorial examples' source code is on github, and different parts of tutorial examples use different branches, so we hope you have a basic knowledge of github before reading this tutorial, and this can help you make better use of this tutorial.

* This tutorial uses a real-time chat application as an example, and we make some modifications of the example to show different features of pomelo, allowing users to have a general understanding of pomelo, and be familiar with it and be able to use it for application development.

* This tutorial assumes that your development environment is Unix-like system, if you use Windows, we hope you know the corresponding manner, such as some .sh script, and uses a bat file with the same name. This tutorial would not make any special instructions for Windows system.

Why chat?
=================

Pomelo is really a game server framework, but it is essentially a high real-time, scalable, multi-process application framework. In addition to some special parts of the game library in the library section, the rest of the framework can be used for development of real-time web application. And compared with some node.js real-time application frameworks such as derby, socketstream, meteor etc, pomelo is more scalable and adaptable.

Because of the complexity of the game in scene management, client animation, they are not suitable entry level application for the pomelo. Chat application is usually the first application which developers contact with node.js, and therefore more suitable for the tutorial.

Therefore, we have chosen to develop a chat application as a tutorial example and use pomelo to develop a chat application which is inherently multi-process, so we can easily extend the server types and quantities.

Content
==============

In the previous sections, we have seen the HelloWorld project and we should have an initial impression about how to develop applications using pomelo. In this tutorial, we will use pomelo to develope a real-time chat application as an example to show adding a filter, doing route compression, making rpc calls and using protobuf etc. These examples are just for demonstrating features of pomelo, so maybe they are not practical. And we hope that users could be familiar with the use of pomelo features through these examples, and this can give users an intuitive understanding, so that users can use the functionality provided by pomelo by the way of examples. Its general content includes following sections:

* Give a brief explanation for some common terminology in pomelo;
* Analyze the source code of a distributed chat application developed by pomelo, and then let it run up;
* Scale up the chat application;
* Add a filter to achieve the desired function , while learn how to use filter in pomelo;
* Pomelo provides route compression, try to use it in our application while learn how to use it in pomelo;
* Pomelo provides message compression based on protobuf, try to apply it to our project, and learn how to use it in pomelo;
* Add a rpc service which can acquire the current time, of course, this functionality is purely superfluous , here is for demonstration purposes;
* Add a HelloWorld component, ibid purpose, just to demonstrate how to extend component in pomelo , and thus learn how to develop custom component;
* Add an admin-module, to demonstrate how to use the admin-module to extend server management control framework of pomelo;
* Finally, summary of pomelo.

Upon completion of the above tutorial, we believe you have been able to skillfully use the various features provided pomelo to do some development, and this is to achieve the purpose of this tutorial. More in-depth discussion can be found in developer's Guide, or directly read the source code. 

[Next, we will explain some of the terminology in pomelo] (Terminologies).