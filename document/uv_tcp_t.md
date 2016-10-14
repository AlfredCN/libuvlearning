表示tcp流或者server,uv_stream_t的子类

## 数据类型

**`uv_tcp_t`**
 tcp handle 类型

## API

**`int uv_tcp_init(uv_loop_t* loop, uv_tcp_t* handle)`**

 初始化handle,这个时候socket还没有创建

**`int uv_tcp_init_ex(uv_loop_t* loop, uv_tcp_t* handle, unsigned int flags)`**

使用特定的选项初始化handle,目前的版本中只使用了最低8位字节表示socket的域(domain).如果设定了域,会根据给定的域创建一个socket.否则如果域设定为AF_UNSPEC则会和uv_tcp_init()一样并不会创建socket.

该接口为1.7.0版本新加.

**`int uv_tcp_open(uv_tcp_t* handle, uv_os_sock_t sock)`**
打开文件描述符或者windows上SOCKET作为TCP handle

1.2.1版本开始: 文件描述符默认设置为非阻塞模式
>接口并没有检查传递的文件描述符或者SOCKET,但是必须保证是有效的流socket.

**`int uv_tcp_nodelay(uv_tcp_t* handle, int enable)`**

开启TCP_NODELAY.

**`int uv_tcp_keepalive(uv_tcp_t* handle, int enable, unsigned int delay)`**

开启/关闭TCP keep-alive. 单位秒,为0的时候忽略

**`int uv_tcp_simultaneous_accepts(uv_tcp_t* handle, int enable)`**

开启/ 关闭 同时异步accept请求,系统内核在监听新tcp连接的时候会把这些请求排队.

这个设置可以优化TCP服务器性能,可以有效提高accept的速度,因此默认是开启的.但是在多进程架构启动的时候有可能导致负载不均衡

**`int uv_tcp_bind(uv_tcp_t* handle, const struct sockaddr* addr, unsigned int flags)`**

绑定ip/端口到handle, addr必须指向已经初始化了的struct sockaddr_in 或 struct sockaddr_in6.

端口已经在使用的话,调用uv_tcp_bind(), uv_listen() 或者 uv_tcp_connect()会收到UV_EADDRINUSE错误.因此成功的调用这个接口并不能保证调用uv_listen() 或者uv_tcp_connect() 一样会成功.

flags可以包括UV_TCP_IPV6ONLY, 这种情况下双栈支持被禁止,只是用IPV6模式

**`int uv_tcp_getsockname(const uv_tcp_t* handle, struct sockaddr* name, int* namelen)`**

获得handle绑定的sockaddr. addr 必须有足够的内存,推荐使用struct sockaddr_storage 来支持IPv4 和 IPv6.

**`int uv_tcp_getpeername(const uv_tcp_t* handle, struct sockaddr* name, int* namelen)`**

获取handle对端的sockaddr,addr 必须有足够的内存,推荐使用struct sockaddr_storage 来支持IPv4 和 IPv6.

**`int uv_tcp_connect(uv_connect_t* req, uv_tcp_t* handle, const struct sockaddr* addr, uv_connect_cb cb)`**

建立IPv4 / IPv6 TCP 连接. 要提供一个已经初始化的TCP handle和未初始化的uv_connect_t. addr要指向有效的sockaddr_in 或者 sockaddr_in6.

回调会在连接建立成功或者出错的时候调用.
