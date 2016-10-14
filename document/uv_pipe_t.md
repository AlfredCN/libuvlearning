Pipe实现unix上的本地域socket和windows上的命名管道.

uv_pipe_t是uv_stream_t的子类

## 数据类型

**`uv_pipe_t`**
Pipe 类型

## API

**`int uv_pipe_init(uv_loop_t* loop, uv_pipe_t* handle, int ipc)`**

pipe 初始化. ipc参数是布尔值,表示pipe时候再进程间传递.

**`int uv_pipe_open(uv_pipe_t* handle, uv_file file)`**

打开一个现有的文件描述符或者HANDLE作为pipe.1.2.1版本开始文件描述符设置成非阻塞模式.
>要确保传递的文件描述符或者HANDLE是有效的pipe,因为接口本身并没有做check

**`int uv_pipe_bind(uv_pipe_t* handle, const char* name)`**
绑定到一个文件路径(unix)或名字(windows)

>unix上的文件路径会被截断到sizeof(sockaddr_un.sun_path) 字节,通常在92 and 108 字节.

**`void uv_pipe_connect(uv_connect_t* req, uv_pipe_t* handle, const char* name, uv_connect_cb cb)`**

连接到unix域socket或者命名管道

>unix上的文件路径会被截断到sizeof(sockaddr_un.sun_path) 字节,通常在92 and 108 字节.

**`int uv_pipe_getsockname(const uv_pipe_t* handle, char* buffer, size_t* size)`**

获得pipe关联的域sockt/命名管道的名字.

buff必须是已经分配好的, size保存buff的大小并且被设置成最终名字的大小.buff大小不够的话,返回UV_ENOBUFS错误并且buffer.len包含需要的大小.

1.3.0版本开始: 返回的大小不在包括结尾的null字节,并且buff不再加结尾null字节.

**`int uv_pipe_getpeername(const uv_pipe_t* handle, char* buffer, size_t* size)`**

获得pipe连接的域sockt/命名管道的名字.

buff必须是已经分配好的, size保存buff的大小并且被设置成最终名字的大小.buff大小不够的话,返回UV_ENOBUFS错误并且buffer.len包含需要的大小.

1.3.0版本新加.

**`void uv_pipe_pending_instances(uv_pipe_t* handle, int count)`**

设置pipe server在等待连接的时候允许同时挂起的pipe实例个数.

>这个选项只在windows平台有效.

**`int uv_pipe_pending_count(uv_pipe_t* handle)`**
**`uv_handle_type uv_pipe_pending_type(uv_pipe_t* handle)`**
在ipc管道上接收handle

首先调用uv_pipe_pending_count(), 如果 > 0 之后初始化uv_pipe_pending_type()返回的pipe并调用uv_accept(pipe, handle).