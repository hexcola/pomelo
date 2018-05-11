Pomelo is based on Node.js, now it has been full supported for Windows, Linux, Mac, etc. 

Prerequisites
=============

* During the process of installation, it will download dependency npm modules for pomelo from the Internet via npm, so make sure your machine can access Internet.

* Make sure your system has installed Node.js. You can install Node using the latest precompiled binary package, Node provides the binary for Windows, Mac and Linux, [here](http://nodejs.org/download/ "Download Node installation package") is the download linkage. You can also use the traditional way to install Node from source code, but it may be more complex. 

* Make sure your system has installed python (2.5 < version < 3.0) and a C++ compiler. The Node's source code is mainly written in C++ and JavaScript, but it use [gyp](http://code.google.com/p/gyp/ "gyp") tool to do the project management, and gyp is written in Python, so python is required. For non-Windows platform, python has been pre-installed generally, so does C++ compiler; But for Windows, make sure your Windows has installed the source code compiler, Visual Studio is recommended. Node.js uses gyp to generate Visual Studio Solution File, and then use the VC++ to compile it to binary.

* Pomelo is written in Javascript, but there are some pomelo dependencies written in C++, so the installation of pomelo will use the C++ compiler. Therefore, please make sure the following two conditions be met if you are on Windows:
    - [Python](http://python.org/) (2.5 < version < 3.0).
    - VC++ compiler, included in the [Visual Studio 2010](http://msdn.microsoft.com/en-us/vstudio/hh388567) (VC++ 2010 Express also be ok).

Additionally, for Windows 8 & 8.1:
* REMOVE Visual Studio C++ 2010, or Visual Studio C++ 2010 Express if installed (In Control Panel > Programs & Features).
* Install Microsoft Visual Studio C++ 2012 for Windows Desktop (The free Express version works well). Be sure to use the "2012" version specifically. This supports both 32-bit and 64-bit processors (x32 and x64).

Additionally, for Windows XP/Vista/7:
* Install Microsoft Visual Studio C++ 2010 (The free Express version works well). NOTE: Windows 8 and 8.1 users, DO NOT install Visual Studio C++ 2010.
* If your processor is x64 (most newer Intel and AMD processors), install the Windows 7 64-bit SDK too.
* Also, if you still get errors that the 64-bit compilers are not installed you may also need the compiler update for the Windows SDK 7.1
* For **Mac**, you should install [Xcode Command Line Tools](https://developer.apple.com/downloads/index.action?q=xcode) or full [Xcode](https://developer.apple.com/xcode/) and **make** firstly.

Additionally, for Windows 10:
* Make sure Microsoft Visual Studio C++ 2010 or 2012 (or Express version) is installed, even if VS2015 is installed which it's not supported yet.


Installation 
================

Using npm (node package management tool) to install pomelo globally:

    $ npm install pomelo -g

You can also download the source code to install pomelo using the following command:

    $ git clone https://github.com/NetEase/pomelo.git
    $ cd pomelo
    $ npm install -g

Where -g option means the global installation. For more infomation on npm, please refer to [npm documentation](https://npmjs.org/doc/ "npm Documents"). 

If the installation does not report any error, that means you have a successful installation.

Next, we will test our installation was successful or not by a [HelloWorld project](Helloworld-of-pomelo "HelloWorld").
