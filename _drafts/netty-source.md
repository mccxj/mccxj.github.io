Netty是一个Java高性能的网络框架，自己也在项目中做过一些尝试，有一定的实践经验。本次阅读源代码，通过结合网上的一些架构设计文章，加深对Netty框架设计的理解。

学习总要带着问题，这样才能有更多收获。这个是关键，所以这一次也不例外。

EventLoop是很关键的设计，本身是一个线程不停地跑，从队列中获取事件，包括nio中的读写操作，对于读的事件，就会触发handler的inbound链的执行，如果是写的事件，那么会？
EventLoopGroup 是一组EventLoop,每个channel生成的时候都会绑定其中一个EventLoop,handler绑定的时候也是如此。这样，默认所有的handler都是同样一个线程处理的。

EventExecutorGroup是一个ScheduledExecutorService，但同时也是Iterable，特定是一个Group,包括多个EventExecutor组(next方法），通过EventExecutor来执行Event逻辑(见AbstractEventExecutorGroup实现)。

再往下的MultithreadEventExecutorGroup实现，要确定2个事情:

如何选择一个EventExecutor 轮训方式

Netty的EventLoopGroup是个什么概念?

ServerSocketChannel初始化流程

SocketChannel的生命周期

线程模型是怎样?

Encode、Decode的主流程

Future/Promise/Listener的应用

这套机制是异步编程的通用模型，Netty也实现了这套机制，为了看懂代码尤为重要。

假设要做的事情叫Task，参与的角色有Task提交者(用户，关注Task结果)，Task管理者(接收Task提交，进行Task调度)，Task执行者(真正干活的)
Future是jdk默认就有的，表示未完成或已经完成的任务的状态(包括成功与否，结果)，用户提交任务以后，就可以得到一个Future对象，后续通过Future获取结果。

用户可以在Future中添加Listener，这样当任务完成之后，就会自动触发回调。

任务调度会生成一个Promise交给执行者，Promise是用来反馈执行结果的。执行者根据执行情况，通过Promise进行反馈(成功，或失败)。

Netty大量使用了这套机制来进行异步编程。

Pipeline 流水线是很关键的类，默认实现是DefaultPipeline, 每个channel都关联pipeline,是线程安全的，内部结构是一个链表，有一个固定的head和tail节点，所有的handler都是在这个链表上，eventloop检测到read事件的时候，会触发fireread事件，进而触发inbound,会从pipeline链表的head开始，选择outbound进行处理。

attr的用途
pipeline是什么?
ChannelHandler单例的情况
ChannelHandler默认名字的生成
channel的ChannelHandler列表如何和一个特定的EventExecutor绑定上
判断当前线程是否inEventLoop?
事件的调用链机制ctx.fireChannelRegistered();
设计模式
ctx， ctx.channel的写模式
