pool handle用来监控文件描述符的读写,断开类似[pool2](http://linux.die.net/man/2/poll)的作用.

poll handle的目的是和第三方的库集成,使得有关socket状态的变化可以通知到event loop,比如c-ares或者libssh2.因为任何其他的目的使用poll都是不推荐的.

uv_tcp_t,uv_udp_t,等相比较uv_poll_t提供了更快速更可扩展的实现,尤其是在windows平台

poll handle通知一个文件描述符可读或者可写但是实际并不能读写的情况是有可能的存在的,所以必须总是检查EGAIN错误

在同一个socket上存在多个激活的poll handle是不可行的,这可能导致libuv出现故障.

poll handle在激活的时候,文件描述符不能被关闭,这会导致handle报告一个错误,或者监听另外一个socket,socket可以在uv_poll_stop()或者uv_close()调用之后立刻被关闭

>在windows平台上,只能监听socket,在unix平台上poll(2)可以接受的文件描述符都可以使用

>在AIX上不支持监听disconnect

## 数据类型
```c
uv_poll_t
Poll handle 类型.

void (*uv_poll_cb)(uv_poll_t* handle, int status, int events)
uv_poll_start()的callback定义.

uv_poll_event
Poll 事件类型

enum uv_poll_event {
    UV_READABLE = 1, //读
    UV_WRITABLE = 2, //写
    UV_DISCONNECT = 4//断开
};
```

## API

`int uv_poll_init(uv_loop_t* loop, uv_poll_t* handle, int fd)`

    使用文件描述符初始化poll
   //1.2.2版本开始文件描述符被设置成非阻塞模式

`int uv_poll_init_socket(uv_loop_t* loop, uv_poll_t* handle, uv_os_sock_t socket)`

    使用socket描述符初始化poll,在类unix系统上和uv_poll_init相同,在windows平台上接受一个SOCKET handle作为参数.
    //1.2.2版本开socket描述符被设置成非阻塞模式


`int uv_poll_start(uv_poll_t* handle, int events, uv_poll_cb cb)`

  开始轮询文件描述符. event由事件类型按位组合,只要检测到事件,callback立即被调用,status参数设置为0,events设置为检测到事件.

 UV_DISCONNECT事件是有可能不被报告出来,用户可以忽略改事件, 但是它可以帮助优化关闭路径(shutdown path),因为可以避免不必要的读写操作.

 轮询的时候发生错误的话,status 会设置成负数并设置成UV_E\*的错误代码.handle在激活状态的时候不能够关闭socket. 如果用户做了关闭操作,callback可能会报告错误状态,但是这是不能保证的
    //1.2.2版本开socket描述符被设置成非阻塞模式
>  在已经激活的hand了上重复多次调用uv_poll_start()是可以的.这样做为更新在监听的事件

`int uv_poll_stop(uv_poll_t* poll)`

  停止轮询文件描述符,回调不会再执行

