# 错误处理
libuv中错误编号是负整数,作为一个规则,只要有status参数的接口或者返回整数的API函数,负整数都代表一个错误,有callback做参数的函数在返回错误的时候callback不会被调用.
> 实现细节:unix系统中错误编号是errno的负数(-errno),在windows系统中被libuv定义成任意的负数

# 错误常量
**UV_E2BIG**
参数列表太长

**UV_EACCES**
没有权限

**UV_EADDRINUSE**
地址已经在使用中

**UV_EADDRNOTAVAIL**
地址不可用

**UV_EAFNOSUPPORT**
不支持的地址族

**UV_EAGAIN**
资源暂时不可用

**UV_EAI_ADDRFAMILY**
不支持的地址族

**UV_EAI_AGAIN**
暂时失败

**UV_EAI_BADFLAGS**
错误的ai_flags值

**UV_EAI_BADHINTS**
错误的hints值

**UV_EAI_CANCELED**
请求被取消

**UV_EAI_FAIL**
永久失败

**UV_EAI_FAMILY**
不支持的ai_family

**UV_EAI_MEMORY**
内存越界

**UV_EAI_NODATA**
没数据(no address)

**UV_EAI_NONAME**
不知道的node或服务

**UV_EAI_OVERFLOW**
参数缓冲区越界

**UV_EAI_PROTOCOL**
未知协议

**UV_EAI_SERVICE**
服务对当前的socket类型无效

**UV_EAI_SOCKTYPE**
不支持的socket类型

**UV_EALREADY**
已经在建立连接

**UV_EBADF**
错误的文件描述符fd

**UV_EBUSY**
资源忙或被锁定

**UV_ECANCELED**
操作取消

**UV_ECHARSET**
无效的unicode字符

**UV_ECONNABORTED**
软件导致的连接失败

**UV_ECONNREFUSED**
连接被拒绝

**UV_ECONNRESET**
连接被对端重置

**UV_EDESTADDRREQ**
需要目标地址

**UV_EEXIST**
文件已经存在

**UV_EFAULT**
在系统调用的参数中地址错误

**UV_EFBIG**
文件太大

**UV_EHOSTUNREACH**
目标(host)无法到达

**UV_EINTR**
系统调用中断

**UV_EINVAL**
无效参数

**UV_EIO**
I/O错误

**UV_EISCONN**
socket已经连接

**UV_EISDIR**
在目录上非法的操作

**UV_ELOOP**
太多的符号链接

**UV_EMFILE**
打开太多文件

**UV_EMSGSIZE**
消息长度太长

**UV_ENAMETOOLONG**
名字太长

**UV_ENETDOWN**
网络down掉了

**UV_ENETUNREACH**
网络无法到达

**UV_ENFILE**
文件表溢出

**UV_ENOBUFS**
没有剩余的缓存空间

**UV_ENODEV**
没有这样的设备

**UV_ENOENT**
没有这个文件或目录

**UV_ENOMEM**
没有足够的内存

**UV_ENONET**
机器不再网络中

**UV_ENOPROTOOPT**
协议不可用

**UV_ENOSPC**
设备上没有剩余空间
no space left on device

**UV_ENOSYS**
函数未实现

**UV_ENOTCONN**
socket未连接

**UV_ENOTDIR**
不是一个目录

**UV_ENOTEMPTY**
不是空目录

**UV_ENOTSOCK**
在非socket上使用socket操作

**UV_ENOTSUP**
在socket上操作不支持

**UV_EPERM**
操作不允许

**UV_EPIPE**
破坏的管道

**UV_EPROTO**
协议错误

**UV_EPROTONOSUPPORT**
不支持的协议

**UV_EPROTOTYPE**
socket上错误的协议

**UV_ERANGE**
返回值太大

**UV_EROFS**
只读文件系统

**UV_ESHUTDOWN**
不能在socket关闭后发送

**UV_ESPIPE**
无效的seek

**UV_ESRCH**
没有这个进程

**UV_ETIMEDOUT**
连接超时

**UV_ETXTBSY**
文本文件忙

**UV_EXDEV**
不允许跨设备建立连接

**UV_UNKNOWN**
未知错误

**UV_EOF**
文件结尾

**UV_ENXIO**
没有该设备或者地址

**UV_EMLINK**
太多连接

# API

const char \* **uv_strerror**(int err)
返回对应错误代码的错误信息

const char\* **uv_err_name**(int err)
返回错误代码的名字