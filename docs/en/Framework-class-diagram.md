In this section, a informal class diagram of the pomelo framework is presented.

There are some notices about the class diagram:

* Actually, since there is not a concept of class but simulative pseudo-class in javascript, so classes within the class diagram is not corresponding to the pseudo-classes defined within the pomelo framework strictly. For example, HandlerFilter, in our class diagram , it is an abstract class to be as the super for all HandlerFilter class, however there is not a corresponding definition in the pomelo framework. We often define just before method and after method for an object and then treat it as a HandlerFilter, without being concerned about their inheritance hierarchy.

* In the class diagram, we name some classes with prefix "Co", in order to indicate that it is a component for pomelo framework.

* In the class diagram, There is a RawSocket class, which is treated as an abstract socket for communication. In practical, it may be WebSocket, TCP or socket.io. All the classes related to RawSocket will participante network communication, including communication with clients or with other servers via RPC invocation.

Here is the class diagram :

![pomelo Framework class diagram](images/pomelo.png)

The following is a brief analysis of the class diagram:

* The Core of the framework is two classes, **Pomelo** and **Application**, Application instance is created by Pomelo. Pomelo consists of a series of components and a global context "Application".

* All the components that have a "Co"-prefixal name presented in the diagram are subclasses of abstract class **Component**. Each component will do its particular work, and be loaded by different servers. The components presented in the class diagram are all built-in component of pomelo framework, which provide core functionalities of pomelo. Developer can customize their own component to extend pomelo framework as well.

* In order to demonstrate pomelo framework better, the classes presented is not only in pomelo framework, but also some classes that is related to the pomelo framework, such as some classes in the [pomelo-admin](https://github.com/NetEase/pomelo-admin) and [pomelo-rpc](https://github.com/NetEase/pomelo-rpc).

* It can be seen from the class diagram that there are many classes that are communication-related as they are related to **RawSocket**, such as MasterAgent & MonitorAgent; MailBox & Acceptor and Connector. In fact , MasterAgent & MonitorAgent are used in server management framework, MasterAgent acts as "server", and MonitorAgent acts as "client". MasterAgent listens on its configured port, MonitorAgent will connects and communicates to MasterAgent, and transfer the commands and monitoring info between them to complete the job for server cluster management, and MailBox and Acceptor do the same work as MasterAgent and MonitorAgent do, but they work for RPC framework. Connector is used to accept connections from clients.

