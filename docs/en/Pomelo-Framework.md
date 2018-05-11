Pomelo框架旨在帮助游戏开发者快速建立游戏底层模型，让其更专注于游戏的业务逻辑开发，具有强大的功能，并且灵活可扩展，这些优势与pomelo框架的设计思想是密不可分的。在参照bigworld, reddwarft等成熟游戏框架优秀设计的基础上，结合以往游戏开发经验，确定了pomelo框架的设计思想，下面就对pomelo框架设计思想的核心部分进行阐述。

#概述
在游戏服务器端，往往需要处理大量的各种各样的任务，比如：管理客户端的连接，维护游戏世界端状态，执行游戏的逻辑等等。每一项任务所需的系统资源也可能不同，如：IO密集或CPU密集等。而这些复杂的任务只用一个单独的服务器进程是很难支撑和管理起来的。所以，游戏服务器往往是由多个类型的服务器进程组成的集群。每个服务器进程专注于一块具体的服务功能，如：连接服务，场景服务，聊天服务等。这些服务器进程相互协作，对外提供完整的游戏服务。

由于存在着上述的这些复杂性，游戏服务器端的开发者往往需要花费大量的时间精力在诸如服务器类型的划分，进程数量的分配，以及这些进程的维护，进程间的通讯，请求的路由等等这些底层的问题上。而这些其实都是一些重复而繁琐的工作，完全可以由更专业，更可靠的框架来抽象和封装，从而将上层的游戏开发者解放出来，把精力更多的放在游戏逻辑的实现上面。Pomelo则是一个为了这个目的而生的框架。

从功能职责上来看，pomelo框架结构如下图所示：
<center>
![server arch](http://pomelo.netease.com/resource/documentImage/pomelo-arch.png)
</center>

* server manager 部分主要负责服务器类型管理，各个进程的创建，管理和监控等。
* network 部分则是负责底层的网络通讯管理，是客户端和服务器之间，各个服务器进程之间通讯的桥梁。同时，network还对底层的细节进行了统一的封装和抽象，对上提供了诸如RPC，channel等基础设施。
* application 部分则是负责每一个进程的具体管理。Application是服务进程的驱动者，是服务进程运作的总入口。同时application也负责维护服务进程的上下文，以及具体游戏服务的配置和生命周期管理等。

下面将围绕着这三部分内容来介绍pomelo框架的设计细节。

#服务器类型
Pomelo框架提供了一套灵活，快捷的服务器类型系统。通过pomelo框架，游戏开发者可以自由地定义自己的服务类型，分配和管理进程资源。
在pomelo框架中，根据服务器的职责不同，服务器主要分为frontend和backend两大类型。二者的关系如下图所示：

<center>
![server type](http://pomelo.netease.com/resource/documentImage/server-type.png)
</center>
 
其中，frontend负责承载客户端的连接，与客户端之间的所有请求和响应包都会经过frontend。同时，frontend也负责维护客户端session并把请求路由给后端的backend服务器。Backend则负责接收frontend分发过来的请求，实现具体的游戏逻辑，并把消息回推给frontend，再最终发送给客户端。在这两类服务器的基础上，可以派生出各种需要的服务器类型，继而生成各种类型的服务器节点。

其实无论frontend也好，backend也好，从上层来看都是一个服务的容器。开发者可以根据容器的职责特性来选择将代码填充到不同的容器中。例如：可以把处理连接的代码放到frontend容器中以获得提供连接服务的服务器类型，而把场景、聊天等代码放到backend容器中获得提供场景服务和聊天服务的服务器类型，如下图所示。也就是说，frontend和backend对外提供什么服务，完全由开发者填充的代码决定。这样，开发者只需要把精力集中在两件事情上：游戏服务器全局的节点分配，以及每个节点上填充的业务代码即可。配置完成后，由Pomelo框架负责将各个服务节点启动并管理起来。

<center>
![container](http://pomelo.netease.com/resource/documentImage/container.png)
</center>
 
#请求/响应流程

##请求和响应
游戏服务器的一种驱动源就是客户端发起请求，服务器端进行处理并响应，也就是典型的请求/响应模式。在pomelo中，客户端可以向服务器发送两种类型的消息：request和notify。

Request消息，包含上行和下行两个消息，服务器处理后会返回响应，pomelo框架会仔细维护好请求和响应之间的对应关系；而notify则是单向的，是客户端通知服务器端的消息，服务器处理后无需向客户端返回响应。请求到达服务器后，先会到达客户端所连接的frontend服务器，后者会根据请求的类型和状态信息将请求分发给负责处理该请求的backend服务器，整个流程如下图所示：

<center>
![request and response](http://pomelo.netease.com/resource/documentImage/request-and-response.png) 
</center>

##请求处理流程
当请求路由到目标服务节点后，将进入请求的处理流程。在pomelo中，请求的处理逻辑主要由游戏开发者完成。请求的处理代码根据职责划分为两大部分：handler和filter。与游戏业务逻辑相关的代码放在handler中完成；而业务逻辑之外的工作，则可以放在filter中。Filter可以看成是请求流程的扩展点。单个服务器上，请求的处理流程如下图所示：
 
<center>
![request flow](http://pomelo.netease.com/resource/documentImage/request-flow.png)
</center>

Filter分为before和after两类，每类filter都可以注册多个，并按注册的顺序出现在请求的处理流程上。

###Before filter
请求首先会先经过before filter。Before filter主要负责前置处理，如：检查当前玩家是否已登录，打印统计日志等。Before filter的接口声明如下：

```javascript
	filter.before = function(msg, session, next)
```

其中，msg为请求消息对象，包含了客户端发来的请求内容；session为当前玩家的会话对象，封装了当前玩家的状态信息；next是进入下一环节的回调函数。

Before filter中调用next参数，流程会进入下一个filter，直到走完所有的before filter后，请求会进入到handler中。另外，也可以通过向next传递一个error参数，来表示filter中处理出现需要终止当前处理流程的异常，比如：当前玩家未登录，则请求的处理流程会直接转到一个全局的error handler（稍后会介绍）来处理。

###Handler

Before filter之后是handler，负责实现业务逻辑。Handler的接口声明如下：

```javascript
	handler.methodName = function(msg, session, next)
```

参数含义与before filter类似。Handler处理完毕后，如有需要返回给客户端的响应，可以将返回结果封装成js对象，通过next传递给后面流程。如果handler处理过程中出现异常，也可以像before filter一样处理，向next传递一个error参数，进入error handler处理。

###Error Handler

Error handler是一个处理全局异常的地方，可以在error handler中对处理流程中发生的异常进行集中处理，如：统计错误信息，组织异常响应结果等。Error handler函数是可选的，如果需要可以通过
```javascript
app.set('errorHandler', handleFunc);
```
来向pomelo框架进行注册，函数声明如下：

```javascript
	errorHandler = function(err, msg, resp, session, next)
```
其中，err是前面流程中发生的异常；resp是前面流程传递过来，需要返回给客户端的响应信息。其他参数与前面的handler一样。

###After filter

无论前面的流程处理的结果是正常还是异常，请求最终都会进入到after filter。After filter是进行后置处理的地方，如：释放请求上下文的资源，记录请求总耗时等。After filter中不应该再出现修改响应内容的代码，因为在进入after filter前响应就已经被发送给客户端。

After filter的接口声明如下：

```javascript
	filter.after = function(err, msg, session, resp, next)
```

参数含义与error handler的一样。After filter中原则上不应再出现流程控制的逻辑，只需要在完成后置工作后，通过next把前面传递过来的结果继续传递下去即可。

经过after filter链后，如果resp不为空，则该响应会传递到玩家客户端所在的frontend服务器并发送到客户端上。至此，整个请求的处理流程完毕。

###Session

Session可以看成一个简单的key/value对象，主要作用是维护当前玩家状态信息，比如：当前玩家的id，所连的frontend服务器id等。Session对象由客户端所连接的frontend服务器维护。在分发请求给backend服务器时，frontend服务器会克隆session，连同请求一起发送给backend服务器。所以，在backend服务器上，session应该是只读的，或者起码只是本地读写的一个对象。任何直接在session上的修改，只对本服务器进程生效，并不会影响到该玩家的全局状态信息。如需修改全局session里的状态信息，需要调用frontend服务器提供的RPC服务。

#channel和广播
##Channel的作用

在游戏服务器端，经常会遇到需要大量广播消息的场景。比如：玩家test在场景中从A点移动到了B点，我们需要把这个信息广播给附近的玩家，这样大家才能看到玩家test移动的效果。于是我们需要有一个能将消息推送给客户端的途径。而channel则是提供这么一个途径的工具。

Channel是服务器端向客户端推送消息的通道。Channel可以看成一个玩家id的容器，通过channel接口，可以把玩家id加入到channel中成为当中的一个成员。之后向channel推送消息，则该channel中所有的成员都会收到消息。Channel只适用于服务器进程本地，即在服务器进程A创建的channel和在服务器进程B创建的channel是两个不同的channel，相互不影响。

##Channel的分类

Pomelo中提供两类channel：具名channel和匿名channel。

具名channel创建时需要指定名字，并会返回一个channel实例。之后可以向channel实例添加、删除玩家id以及推送消息等。Channel实例不会自动释放，需要显式调用销毁接口。具名channel适用于需要长期维护的订阅关系，如：聊天频道服务等。

匿名channel则无需指定名字，无实例返回，调用时需指定目标玩家id集合。匿名channel适用于成员变化频率较大、临时的单次消息推送，如：场景AOI消息的推送。

两种channel对上层的表现形式不一样，但底层的推送机制是相似的。Channel的推送过程分为两步：第一步从channel所在的服务器进程将消息推送到玩家客户端所连接的frontend进程；第二步则是通过frontend进程将消息推送到玩家客户端。第一步的推送的实现主要依赖于底层的RPC框架（下一节中介绍）。推送前，会根据玩家所在的frontend服务器id进行分组，一条消息只会往同一个frontend服务器推送一次，不会造成广播消息泛滥的问题。

<center>
![channel](http://pomelo.netease.com/resource/documentImage/channel.png)
</center>

#RPC框架
##RPC的作用

从前面的介绍可以知道，在pomelo中，游戏服务器其实是一个多进程相互协作的环境。各个进程之间通讯，主要是通过底层统一的RPC框架来完成。

在pomelo的RPC框架中，主要考虑解决以下两个问题。

第一个问题是进程间消息的路由策略。由前面的游戏场景分区策略中可以看出，游戏服务是有状态的服务，玩家总与某个场景相关联，场景需要记录玩家在场景中的状态，而相关的请求也必须路由到玩家所在的场景服务中。所以，服务器端的消息路由不单与请求的类型相关，也与玩家的状态相关。比如：玩家A在场景1中移动，则移动的请求应该发往场景1所在的服务进程；而玩家A传送到场景2后，同样的移动请求则需要路由到场景2所在的进程。然而，不同的游戏，状态信息可能会不同，路由的规则也不尽相同。所以，Pomelo框架需要提供一个灵活的机制，让开发者能自由的根据玩家状态控制消息的路由。

第二个问题是RPC底层通讯协议的选择。不同的游戏对服务器之间的通讯协议要求可能也不一样，有的情况可能需要tcp，有的时候可能udp就可以了，再或者需要对传输的数据进行一些加工，如probuffer之类。所以也需要提供一个机制来给开发者来选择和定制他们所需的底层通讯协议。

Pomelo的RPC框架中，引入了多层抽象来解决上述的两个问题。

##RPC客户端

在RPC客户端，层次结构如下图所示：

<center>
![rpc client](http://pomelo.netease.com/resource/documentImage/rpc-client.png)
</center>
 
在最底层，使用mail box的抽象隐藏了底层通讯协议的细节。一个mail box对应一个远程服务器的连接。Mail box对上提供了统一的接口，如：连接，发送，关闭等。Mail box内部则可以提供不同的实现，包括底层的传输协议，消息缓冲队列，传输数据的包装等。开发者可以根据实际需要，实现不同的mail box，来满足不同的底层协议的需求。

在mail box上面，是mail station层，负责管理底层所有mail box实例的创建和销毁，以及对上层提供统一的消息分发接口。上层代码只要传递一个目标mail box的id，mail station则可以知道如何通过底层相应的mail box实例将这个消息发送出去。开发者可以给mail station传递一个mail box的工厂方法，即可以定制底层的mail box实例的创建过程了，比如：连接到某个服务器，使用某一类型的mail box，而其他的服务器，则使用另外一个类型的mail box。

再往上的是路由层。路由层的主要工作就是提供消息路由的算法。路由函数是可以从外面定制的，开发者通过注入自定义的路由函数来实现自己的路由策略。每个RPC消息分发前，都会调用路由函数进行路由计算。容器会提供与该RPC相关的玩家会话对象（当中包含了该玩家当前的状态）和RPC的描述消息（包含了RPC的具体信息），通过这两个对象，即可做出路由的决策。路由的结果是目标mail box的id，然后传递给底下的mail station层即可。

最上面的是代理层，其主要作用是隐藏底层RPC调用的细节。Pomelo会根据远程接口生成代理对象，上层代码调用远程对象就像调用本地对象一样。但这里对远程代理对象有两个约定的规则，即第一个参数必须是相关玩家的session对象，如果没有这么一个对象可以填充null，在路由函数中需做特殊处理。还有就是最后一个参数是RPC调用结果的回调函数，调用的错误或是结果全部通过该回调函数返回。而在远程服务的提供端，方法的声明与代理端的声明相比，除了不需要第一个session参数，其余的参数是一样的。

远程服务提供端的方法声明：

```javascript
	remote.echo = function(msg, cb) {
		// …
	};
```

代理端的方法声明：

```javascript
	proxy.echo = function(session, msg, cb) {
		// …
	};
```

Pomelo框架同时也提供了直接调用RPC的入口，如果上层代码明确知道目标的mail box id，可以直接通过这个接口直接把消息发送出去。

##RPC服务提供端

在RPC服务提供端，层次结构相对简单一些，如下图所示：

<center>
![rpc server](http://pomelo.netease.com/resource/documentImage/rpc-server.png)
</center>
 
最底下的是acceptor层，主要负责网络监听，消息接收和解析。Acceptor层与mail box层相对应，可以看成是网络协议栈上同一层上的两端，即从mail box层传入的消息与acceptor层上传出的消息应该是同样的内容。所以这两端的实例必须一致，使用同样的底层传输协议，对传输的数据使用同样格式进行封装。在客户端替换了mail box的实现，则在服务提供端也必须替换成对应的acceptor实现。

往上是dispatch层。该层主要完成的工作是根据RPC描述消息将请求分发给上层的远程服务。

最上层的是远程服务层，即提供远程服务业务逻辑的地方，由pomelo框架自动加载remote代码来完成。

#服务器的扩展

每一个服务进程都维护着一个application的实例app。App除了提供一些基本的配置和驱动接口，更多的是充当着服务进程上下文的角色。开发者可以通过app进行一些上下文共享和扩展工作，从而实现各个服务模块之间的解耦。App实例在handler和remote接口中已通过工厂方法注入，另外也可以通过require('pomelo').app来获取到当前进程唯一的app实例。

##set 与 get
app.set和app.get的语义与一般的set和get操作的语义一样，即往上下文中保存和读取键值对。开发者可以通过get/set的机制，在app中存放一些全局的属性，以及一些无需生命周期管理的服务对象。

##组件
###组件的定义
组件（component）是纳入服务器生命周期管理的服务单元。组件一般以服务为单位来划分，一个组件负责实现一类具体的服务，如：加载handler代码，开启websocket监听端口等。App作为服务器的主干代码，并不会参与具体的服务逻辑，更多的是充当上下文和驱动者的角色。开发者可以定义自己的组件，加入到服务器的生命周期管理中，从而来对服务器的能力进行扩展。服务器启动和关闭流程主要就是通过app驱动各个组件启动和关闭的过程。大致流程如下图所示：

<center>
![components](http://pomelo.netease.com/resource/documentImage/components.png)
</center>

组件可以根据需要，提供不同的生命周期接口，pomelo框架会在生命周期的各个阶段触发相应的回调。组件的生命周期接口类型如下：

* start(cb) 服务器启动回调。在当前服务器启动过程中，会按注册顺序触发各组件的这个接口。组件启动完毕后，需要调用cb函数通知框架执行后续流程。
* afterStart(cb) 服务器启动完毕回调。当pomelo管理下的所有服务器进程都启动完成后会触发这个接口。一些需要等待全局就绪的工作可以放到这里来完成。
* stop(force, cb) 服务器关闭回调。在服务器关闭期间，会根据注册顺序的逆序来触发这个回调接口，主要用来通知各个组件保存各自的数据和释放资源。force表示是否要强制关闭。操作完毕后，同样需要调用cb函数来继续后续流程。

###组件的加载
组件可以通过
```javascript
app.load([name], comp, [opts])
```
接口来完成。

* name 是可选的组件名称。有名字的组件在加载之后可以通过app.components.name来获取。
* comp 是组件对象或组件对象的工厂方法。如果comp是一个函数，则会调用这个函数，同时把app实例和opts当参数传递给这个函数，并把它的返回值做为组件对象。否则直接把comp作为组件对象。
* opts 是一个可选的附加参数，将被传递给comp工厂方法。

# 总结
以上内容详细阐述了pomelo框架服务器类型划分、请求/响应流程、Channel广播机制和RPC框架以及服务器的扩展等核心部分的设计细想和方案。它们很好的解决了服务器的抽象与扩展、请求响应和服务器间通讯等问题，保证了框架的可扩展、灵活易用性及高性能等特性。在此基础上，游戏开发者就可以避免枯燥乏味的底层逻辑和重复劳动，专注于游戏业务逻辑的开发，按照pomelo框架提供的[api文档](http://pomelo.netease.com/api.html)，结合[pomelo快速使用指南](https://github.com/NetEase/pomelo/wiki/pomelo快速使用指南)，[架构概览](https://github.com/NetEase/pomelo/wiki/pomelo%E6%9E%B6%E6%9E%84%E6%A6%82%E8%A7%88)等文档，便可以轻松的进行游戏开发工作。