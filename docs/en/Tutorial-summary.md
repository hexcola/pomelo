Now, the basic core features of pomelo have been presented before by examples. There are also some functionalities not covered in previous tutorial, and we put them here to make a brief description. At last, we make a summary for this whole tutorial.

Not Covered
=============

### Servers Administration

Pomelo provides a command-line tool to help developers develop more conviently and easily. Since pomelo 0.6.0, Username/password is required for authentication while using this tool, its command line format is as follows:
  
    $ pomelo [list|stop|kill] [-u <username>] [-p <password>]

 [pomelo-cli](pomelo-cli-usage) provides more powerful capabilities to monitor and mange the server cluster.

### Plugin 

Pomelo provides a plugin-based extension mechanism, a plugin can contain some related components and a set of event handlers for internal events of pomelo, more detail is presented at [plugin reference documentation] (https://github.com/NetEase/pomelo/wiki/plugin).

### IDE 

While developing, a handy IDE may be required. For pomelo, [here] (https://github.com/NetEase/pomelo/wiki/Debugging-Pomelo-ServerApps-With-WebStorm-IDE) introduces a powerful IDE for javascript.

### Server Configuration

This tutorial talks shallowly on how to configure server information. You can get more about server configuration in section User Manual. 

Summary
=========

By the example chat, we start from the initialization of it, and show some features by change it step by step. we just aim to let the developer know how to add a filter, how to add a rpc invocation, how to enable and use protobuf encoding/decoding, how to customize your own component and let pomelo load it, how to customize an admin-module, etc.. These features are all pomelo core functionalities.

After learning this example, we believe you would have a more detailed understanding of pomelo and its functionalities, as well as how to use pomelo to develop an application. This example can also be used as a sample application for developers, it basically involves all aspects of pomelo.
