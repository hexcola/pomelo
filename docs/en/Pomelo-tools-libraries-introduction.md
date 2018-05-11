Pomelo provides a series of tools and libraries for developers to help developers to develop, debug, and deploy. These tools and libraries provide a large range of functionalities including server management & control, stress testing, also some of libraries that is commonly used in any applications.

### Command-Line Tool of Pomelo

This tool can help developers to develop applications more conveniently and more efficiently, its functionalities include creation of a project , start an application, stop an application, close an application, etc., please refer to [pomelo command-line tool documentation](pomelo-command-line-tool) for more detail .

### Pomelo-cli

pomelo-cli is a client that can be used to manage server cluster, it should connect and register to the master server at first, and then you can send commands to server cluster to do some management, such as starting a server dynamically at runtime, viewing a server's status. Please refer to [pomelo-cli documentation ] (pomelo-cli-usage) for more detail.

### Pomelo-robot

pomelo-robot is a framework used to run benchmark and stress testing, please refer to [pomelo-robot documentation] (pomelo-robot-usage) for more detail.

### Pomelo-daemon

pomelo-daemon provides a daemon service, you can use this service for distributed deployment and log collection. Please refer to [pomelo-daemon documentation] (pomelo-daemon-usage) for more detail.

### Pomelo-admin-web

pomelo-admin-web is a web client to monitor server cluster based on [pomelo-admin] (https://github.com/NetEase/pomelo-admin), you can use it to monitor running status, performance, logs and other information of server cluster through a browser. Please refer to [pomelo-admin-web tool documentation](pomelo-admin-web-tool-usage) for more detail.

### Pomelo-sync

pomelo-sync is a module that used to manage synchronization for the data that should be persistented between memory and external storage systems. Please refer to [pomelo-sync documentation](pomelo-sync-usage) for more detail.

### Pomelo-protobuf

pomelo-protobuf is a variant implementation for [google protobuf](http://code.google.com/p/protobuf/) written in javascript language, and using a json file to substitute ".proto" file. It parses the json file at runtime dynamicly, avoiding compiling .proto file. It is used to pack the communicate message in pomelo framework more compactly. Please refer to [pomelo-protobuf] (https://github.com/pomelonode/pomelo-protobuf) for more detail.
