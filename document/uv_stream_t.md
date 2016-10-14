stream handle 提供一个双工通信频道的抽象,uv_stream_t是抽象类型,libuv提供3种流的实现:uv_tcp_t,uv_pipe_t,uv_tty_t

## 数据类型

**`uv_stream_t`**

stream handle 类型

**`uv_connect_t`**

connect request类型

**`uv_shutdown_t`**

shutdown request类型

**`uv_write_t`**

write request类型.重用write对象的时候必须特别注意,当一个流在非阻塞模式情况下,通过uv_write发送的request会被排队,这个时候对象被重用会导致未定义的行为.只有在uv_write的回调执行之后,重用uv_write_t对象才是安全的

**`void (*uv_read_cb)(uv_stream_t* stream, ssize_t nread, const uv_buf_t* buf)`**

数据读到缓冲区之后被调用

如果有数据nread > 0,nread < 0 表明出错,当遇到EOF的时候nread会被设置成UV_EOF,出错的时候buf可能指向无效的缓冲区,这种情况下,buf.len以及buf.base全部设置为0,nread可能为0,不表示错误或者EOF,等同于EAGAIN/EWOULDBLOCK

回调函数在出现错误的时候有责任调用uv_read_stop()或者uv_close()关闭流,出错的流上再次调用read是未定义的.
回调函数有责任释放缓冲区,libuv不会重用缓冲区

**`void (*uv_write_cb)(uv_write_t* req, int status)`**

数据写结束之后调用,status 0表示成功,否则status小于0

**`void (*uv_connect_cb)(uv_connect_t* req, int status)`**

uv_connect结束的回调,status 0表示成功,小于0表示失败

**`void (*uv_shutdown_cb)(uv_shutdown_t* req, int status)`**

shutdown结束的时候调用,status 0表示成功,小于0表示失败

**`void (*uv_connection_cb)(uv_stream_t* server, int status)`**

server接受到新的连接的时候调用,uv_accept()来接受新连接.status 0表示成功,小于0表示失败

## 公有成员

**`size_t uv_stream_t.write_queue_size`**

只读,缓冲的等待写出的数据大小

**`uv_stream_t* uv_connect_t.handle`**
**`uv_stream_t* uv_shutdown_t.handle`**
**`uv_stream_t* uv_write_t.handle`**

 执行request的handle

**`uv_stream_t* uv_write_t.send_handle`**

 发送的handle ==TODO??

## API

**`int uv_shutdown(uv_shutdown_t* req, uv_stream_t* handle, uv_shutdown_cb cb)`**

关闭双工流的写入一侧,会等待缓冲的写入请求执行完毕. handle必须是已经初始化的stream,req必须是还没有初始化的request

**`int uv_listen(uv_stream_t* stream, int backlog, uv_connection_cb cb)`**

开始监听新到的连接,backlog表明内核排队的数量,和[listen\(2\)](http://linux.die.net/man/2/listen)一致,有新连接的时候uv_connection_cb回调会被调用

**`int uv_accept(uv_stream_t* server, uv_stream_t* client)`**

和uv_listen配合使用接受新的连接,在收到uv_connection_cb回调之后调用此方法来接受新的连接,调用之前client handle必须已经初始化. 返回值<0的话表示错误

在uv_connection_cb回调执行的时候,uv_accept第一次调用一定会成功.如果你多次调用uv_accept就可能失败,因此建议每次uv_connection_cb回调只调用一次uv_accepet.

> server和client handle 必须在同一个事件循环中

**`int uv_read_start(uv_stream_t* stream, uv_alloc_cb alloc_cb, uv_read_cb read_cb)`**

从输入流中读取数据,read_cb可能会多次执行一直到不再有数据可读或者调用了uv_read_stop()

**`int uv_read_stop(uv_stream_t*)`**

停止从流上读取数据,这个函数可以安全的已经停止读取的流上多次调用

**`int uv_write(uv_write_t* req, uv_stream_t* handle, const uv_buf_t bufs[], unsigned int nbufs, uv_write_cb cb)`**

向流上写入数据,数据按章buffer的顺序写入

```c
void cb(uv_write_t* req,int status){}
uv_buf_t a[]={
{.base="1",.len=1},
{.base="2",.len=1},
}

uv_buf_t b[]={
{.base="3",.len=1},
{.base="4",.len=1},
}

uv_write_t req1,req2;
//写入"1234"
uv_write(&req1,stream,a,2,cb);
uv_write(&req2,stream,b,2,cb);

```

**`int uv_write2(uv_write_t* req, uv_stream_t* handle, const uv_buf_t bufs[], unsigned int nbufs, uv_stream_t* send_handle, uv_write_cb cb)`**
扩展通过pipe发送数据,pipe必须初始化的时候设定ipc=1.

>send_handle必须是一个tcp socket或者pipe,必须是server或者一个连接(处在监听或者已连接的状态).Bound sockets or pipes will be assumed to be servers

**`int uv_try_write(uv_stream_t* handle, const uv_buf_t bufs[], unsigned int nbufs)`**

和uv_write相同,但是不能立刻完成的时候不会把请求排队缓存起来.

返回值:
* >0:写入的字节数,可以小于提供的buff大小 (??TODO这种情况下剩余的数据怎么办??自己多次调用??)
* <0:错误码,如果数据不能立刻被发送返回UV_EAGAIN

**`int uv_is_readable(const uv_stream_t* handle)`**

流可读的话返回1,否则返回0.

**`int uv_is_writable(const uv_stream_t* handle)`**

流可写的话返回1,否则返回0.

**`int uv_stream_set_blocking(uv_stream_t* handle, int blocking)`**

设置阻塞模式.阻塞的时候,所有的写入同步完成,接口并没有改变,写入完成或者失败依然是通过回调通知.

>不建议过多的依赖这个API接口,该接口在之后的版本中可能被修改,目前在windows平台上只支持uv\_pipe\_t,unix平台上所有的strean类型都支持.
>libuv也不保证阻塞模式改变之后,之前已经提交write请求的发送顺序.因此强烈建议在打开或者创建流之后立即设置阻塞模式


