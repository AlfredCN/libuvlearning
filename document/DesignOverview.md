#  设计综述
libuv最初是node.js设计的跨平台的支持库.实现了事件驱动的异步I/O模型.

libuv不止提供了各种不同I/O轮询机制的简单抽象,还通过'handles'和'streams'提供了socket等其他实体的高层抽象,跨平台文件I/O和线程操作等等

下图说明了构成libuv的各个不同组件,以及相关的子系统
![architecture](http://docs.libuv.org/en/v1.x/_images/architecture.png)

# Handles和requests

libuv提供了两种抽象来和事件循环(event loop)协同工作:handles和requests

Handles表示长生存周期的对象,在生命期内执行特定的操作.例如: prepare在每次循环迭代的时候调用它的回调,TCP serverhandle在每次有新连接的时候调用连接的回调函数

Requests通常是短期对象,执行短期操作,这些操作既可以在handle之上执行比如写请求(write request)在一个handle上写数据,也可以独立执比如getaddrinfo 直接在loop上执行

# I/O循环
I/O(事件)循环是libuv的核心部分,所有I/O操作都建立在上面,并且是绑定在一个单独的线程上面.程序可以运行多个事件循环,但是每个循环要运行在不同的线程中.除非注明,libuv的事件循环或者任何其他和loop以及handle相关的API**都不是线程安全**的

事件循环遵循通常的单线程异步I/O方法,所有网络I/O使用所在平台上最优的轮询机制,在非阻塞的socket之上执行,linux上epoll,OSX和BSD系统上使用kqueue,sunOS系统上使用event ports,windows上使用IOCP.

每次循环迭代,loop会阻塞等待socket上面激活的I/O事件,并且触发相应的回调以执行对应的读、写或者其他期望的操作.

为了更好地理解loop的操作,下图说明了一次循环迭代中所有的步骤:
![loop_iteration](http://docs.libuv.org/en/v1.x/_images/loop_iteration.png)
1. 更新当前时间,循环开始的时候,缓存当前的时间以减少时间相关的系统调用.
2. 如果loop在可用状态(alive),开启新的迭代,否则退出循环.loop含有激活并且被引用的handle,激活的request或者正在关闭中的handle都会被认为处在可用状态.
3. 定时器调用,调用到期的定时器回调
4. 调用延迟的回调.大部分情况下,I/O事件发生的时候,对应的回调会立刻被调用,但是某些情况下,回调函数会被延迟到下次循环迭代.这些上次迭代中延迟的回调函数会在这里被调用.
5. 调用idle handle的回调函数,只要idle在激活状态,它的回调函数每次迭代都会执行
6. 调用prepare handle的回调函数,I/O操作可能会阻塞,所以放在io之前调用
7. 计算轮询超时时间.超时时间的计算规则:
    * UV_RUN_NOWAIT标记下运行的话,超时时间0
    * uv_stop()调用的话,超时时间0
    * 没有激活的handle和request,超时时间0
    * 有激活的idle,超时时间0
    * 有要关闭的handle,超时时间0
    * 如果都不匹配,使用最近的定时器的超时时间,如果没有激活的定时器,一直阻塞
8. 阻塞等待I/O.跟文件I/O相关的handle的回调函数在这里被调用
9. 调用check的回调,在I/O操作之后调用,本质上是prepare handle的副本
10. 调用uv_close()的回调.
11. uv_run_once标记运行的话,如果某些定时器到期的话,在这里调用这些定时器的回调
12. 迭代结束.如果loop使用UV_RUN_NOWAIT或者UV_RUN_ONCE运行的话循环结束uv_run返回,如果使用UV_RUN_DEFAULT标记调用,并且在可用状态循环继续否则结束循环返回

> 文件I/O内部使用同步的系统调用,libuv用线程池来实现异步文件I/O,**网络I/O一直在单线程中执行**

> 尽管轮询模型不同但是libuv在unix和windows系统上的执行模型保持一致

# 文件I/O
和网络I/O不同的是,没有libuv可以使用的基础的特定于某个平台的文件I/O库,目前的方法是在线程池中使用阻塞的文件I/O操作

想要彻底的了解跨平台文件I/O的设计,查看[asynchronous-disk-io](http://blog.libtorrent.org/2012/10/asynchronous-disk-io/)

libuv现在使用全局的线程池,所有的loop上的文件I/O在这个线程池上排队工作.目前共有三种类型的操作运行在这个线程池上:
    * 文件系统操作
    * DNS函数(getaddrinfo 和 getnameinfo)
    * 用户通过uv_queue_work()指定的逻辑代码
 >**警告:**更多细节查看[Thread pool work scheduling](http://docs.libuv.org/en/v1.x/threadpool.html#threadpool),但是要注意线程池的大小是有限的