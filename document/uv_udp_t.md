封装客户端和服务器的UDP通信

## 数据类型

**`uv_udp_t`**

udp类型

**`uv_udp_send_t`**
udp 发送请求类型

**`uv_udp_flags`**
 在uv_udp_bind()和uv_udp_recv_cb中的标记

```c
enum uv_udp_flags {
    UV_UDP_IPV6ONLY = 1, //取消双栈模式,只用ipv6
    UV_UDP_PARTIAL = 2,//设置如果读buff太小的话,消息可以被截断,剩余的消息被系统丢弃,用在uv_udp_recv_cb中
    /*
    * 表明在uv_udp_bind的时候设置SO_REUSEADDR选项
    * 在BSDS和OSX系统上设置的是SO_REUSEPORT其他unix平台设置SO_REUSEADDR 
    * 意味着多个线程或进程可以绑定相同的地址不会出错(都设置的改标记),但之后最后一个可以收取数据,引发从之前监听socket上偷取端口的效果
    */
    UV_UDP_REUSEADDR = 4
};
```
**`void (*uv_udp_send_cb)(uv_udp_send_t* req, int status)`**

 uv_udp_send()的回调定义,

**`void (*uv_udp_recv_cb)(uv_udp_t* handle, ssize_t nread, const uv_buf_t* buf, const struct sockaddr* addr, unsigned flags)`**

uv_udp_recv_start()的回调定义,在接收到数据的时候调用.

参数:
* handle: UDP handle
* nread: 已经接收的字节数.0表示已经没有数据可读. 你可以丢弃或者重用读取缓冲区,注意0同时也可能表示收到了空的数据报(这种情况下addr是空指针). < 0表示传输出错
* buf: 用来接收数据的uv_buf_t.
* addr: struct sockaddr*类型的数据包含发送端的地址,可以为NULL,只在callback回调内有效.
* flags: 接收标记目前只使用了UV_UDP_PARTIAL.

>注意: nread= =0 并且 addr = = NULL的时候表示没有数据可读,nread == 0 并且 addr != NULL 表明收到了一个空的数据报.

**`uv_membership`**
组播地址的成员类型.
```c
typedef enum {
    UV_LEAVE_GROUP = 0, //离开组
    UV_JOIN_GROUP       //加入组   
} uv_membership;
```

## 公有成员

**`size_t uv_udp_t.send_queue_size`**

发送队列的大小.这个字段表明有多少信息正在排队发送.

**`size_t uv_udp_t.send_queue_count`**

发送队列上正在等待发送的请求数量.

**`uv_udp_t* uv_udp_send_t.handle`**

发送请求关联的UDPhandle.

## API

**`int uv_udp_init(uv_loop_t* loop, uv_udp_t* handle)`**

初始化UDP,实际的socket是延迟创建的,返回0表示成功.

**`int uv_udp_init_ex(uv_loop_t* loop, uv_udp_t* handle, unsigned int flags)`

使用特性的选项初始化UDP.当前版本只用低8为表示socket域,初始化的过程中会在域上创建新的socket,如果域指定的是AF_UNSPEC和uv_udp_init()一样不会创建socket.

1.7.0版本新加.

**`int uv_udp_open(uv_udp_t* handle, uv_os_sock_t sock)`**

打开现有的文件描述符/SOCKET作为UDP handle.

Unix平台上:只要遵循数据报约定(工作在非连接模式,支持sendmsg()/recvmsg(),等等)的socket就可以.换句话说,原始(raw)socket,netlink socket也支持.

1.2.1版本开始: 文件描述符设置为非阻塞模式.

>接口没有检查传入的socket的类型,必须自己确保传入的有效的 数据报 socket.

**`int uv_udp_bind(uv_udp_t* handle, const struct sockaddr* addr, unsigned int flags)`**

绑定到具体的ip和端口
参数:
* handle – UDP handle. 必须已经使用uv_udp_init()初始化.
* addr   – sockaddr_in 或sockaddr_in6类型地址/端口.
* flags  – 指定socket怎么绑定,目前支持 UV_UDP_IPV6ONLY 和 UV_UDP_REUSEADDR.
  返回:
  0 成功,<0 失败

**`int uv_udp_getsockname(const uv_udp_t* handle, struct sockaddr* name, int* namelen)`**

获取UDP handle的本地IP和端口.

参数:	
* handle –UDP handle. 必须已经使用uv_udp_init()初始化.
* name – name必须有足够的内存,推荐使用struct sockaddr_storage 来支持IPv4 和 IPv6.
* namelen – 输入的时候表示name的长度,输出表示填充了多少字节.
  返回:
  0成功, <0 失败

**`int uv_udp_set_membership(uv_udp_t* handle, const char* multicast_addr, const char* interface_addr, uv_membership membership)`**

设置多播地址的成员

参数:	
* handle – UDP handle.已经初始化.
* multicast_addr – 要设置的多播地址.
* interface_addr – 网络设备接口ip地址,在该设备所在的子网中加入组播组.
* membership – 加入还是删除:UV_JOIN_GROUP/UV_LEAVE_GROUP.
  返回:0成功,<0失败

**`int uv_udp_set_multicast_loop(uv_udp_t* handle, int on)`**

设置IP多播循环标记,使广播包循环训本地socket.

参数:	
* handle – UDP handle.已经初始化.
* on – 1 开启, 0 关闭.
  返回:0 成功, <0 失败

**`int uv_udp_set_multicast_ttl(uv_udp_t* handle, int ttl)`**

设置组播TTL

参数:	
* handle – UDP handle.已经初始化.
* ttl – 1 到 255.
  返回:0 成功, <0 失败

**`int uv_udp_set_multicast_interface(uv_udp_t* handle, const char* interface_addr)`**
设置多播用来发送和接收数据的网络设备接口地址
Set the multicast interface to send or receive data on.

参数:	
* handle – UDP handle.已经初始化.
* interface_addr – 网络设备接口地址.
  返回: 0 成功, <0 失败.

**`int uv_udp_set_broadcast(uv_udp_t* handle, int on)`**

设置是否多播

参数:	
* handle – UDP handle.已经初始化.
* on – 1 开启, 0 关闭.
  返回: 0 成功, <0 失败.

**`int uv_udp_set_ttl(uv_udp_t* handle, int ttl)`**

设置ttl

参数:	
* handle – UDP handle.已经初始化.
* ttl – 1 到 255.
  返回:0 成功, <0 失败

**`int uv_udp_send(uv_udp_send_t* req, uv_udp_t* handle, const uv_buf_t bufs[], unsigned int nbufs, const struct sockaddr* addr, uv_udp_send_cb send_cb)`**

发送UDP数据,如果没有绑定,会绑定到0.0.0.0和一个自动端口.

参数:	
* req – 尚未初始化的请求.
* handle – UDP handle.已经初始化.
* bufs – buff.
* nbufs – buff个数.
* addr – 对端地址.
* send_cb – 发送回调.
  返回:0 成功, <0 失败

**`int uv_udp_try_send(uv_udp_t* handle, const uv_buf_t bufs[], unsigned int nbufs, const struct sockaddr* addr)`**

和uv_udp_send()一样,但是发送请求没有立即完成的情况下不会对请求做排队.

返回: >= 0: 发送的字节, <0 错误, 返回UV_EAGAIN表示没有被立刻发送.

**`int uv_udp_recv_start(uv_udp_t* handle, uv_alloc_cb alloc_cb, uv_udp_recv_cb recv_cb)`**

准备接收数据.sockt没有绑定的话,绑定到0.0.0.0:随机端口.

参数:

* handle – UDP handle.已经初始化.
* alloc_cb – 需要临时内存时候的分配函数.
* recv_cb – 接收数据回调.

返回:0 成功, < 0 失败错误码.

**`int uv_udp_recv_stop(uv_udp_t* handle)`**

停止接收数据报.

参数:	
* handle – UDP handle.已经初始化.
  返回值:0 成功, < 0 失败错误码.
