# uv_hanlde_t handle基类
   *uv_handle_t* 是libuv所有handle类型的基类(C结构体字段对齐)
   结构体对齐(前面的字段保持和uv_handle_t一致)以保证所有handle都可以类型转换到uv_handle_t

# 数据类型
## uv_handle_t
   libuv handle的基类
## uv_any_handle
   所有handle类型构成的union

void (\* **uv_alloc_cb**)(uv_handle_t\* handle,size_t suggested_size,uv_buf_t\* buf)
   传递给**uv\_read_start**和**uv\_udp_recv_start**的缓存分配函数.用户必须分配内存填充uv\_buff_t结构体,如果buff设置成NULL或者长度为0的话,在**uv\_udp_recv_cb**或者**uv\_read_cb**回调调用的时候触发**UV_ENOBUFS**的错误.

**uv\_buf_t**的大小可以是一个大于0的值,不一定非要设置成suggested_size(当前值65536).

void (\* **uv\_close_cb**)(uv\_handle\* handle)
   **uv\_close()**的回调函数定义

## 公有成员

uv\_loop_t\* **uv\_handle_t.loop**
   只读.到所在事件循环的指针

void* uv\_handle_t.data
   用户自定义数据

## <a name="md-api" id="md-api">API</a>
int **uv\_is_active**(const uv\_handle_t\* *handle*)
   激活状态返回非零,0表示不在激活状态,激活状态依赖handle类型
* uv\_async_t 除非调用uv_close()关闭,否则一直处于激活状态
* uv\_pipe_t,uv\_tcp_t,uv\_udp_t...等处理I/O的handle,在做调用IO的的处理时比如读,写,连接,接受新连接等等是激活状态
* uv\_check_t,uv\_idle_t,uv\_timer_t...当调用对应的start函数后是激活状态uv\_check_start(),uv\_idle_start()...

 > 如果handle uv\_foo_t有函数uv\_foo_start()函数,那么该函数调用后handle处在激活状态,同时uv\_foo_stop()调用后在非激活状态

int **uv\_is_closing**(const uv\_handle_t\* *handle*)
 如果handle正在关闭或者已经关闭返回非0值,否则返回0
 > 该函数只能在初始化之后并且close从回调之前使用

void **uv\_close**(uv\_handle_t\* hanlde,uv\_close_cb close\_cb)
  请求关闭handle,close\_cb会异步的当时被调用,并且**必须**在handle内存被释放之前调用
  包装文件描述符的handle会被立刻关闭,但是close\_cb会延迟到下一次循环调用,使你有机会释放handle关联的资源
  正在进行中的请求,例如uv\_connect_t 或者 uv\_write_t会被取消,他们的回调会被异步调用返回状态 UV\_ECANCELED

void **uv\_ref**(uv\_hanlde_t\* *handle*)
  引用给定的handle, 引用时幂等的,也就是说,如果handle已经有引用了,再次调用该函数没有任何效果
  参见:[引用计数](#refcount "引用计数")

void **uv\_unref**(uv\_handle_t\* *handle*)
  handle解引用,如果handle没有被引用调用该函数没有任何效果
  参见:[引用计数](#refcount "引用计数")

int **uv\_has_ref**(const uv\_handle_t* *hanlde*)
  如果handle有引用返回非0,否则返回0
  参见:[引用计数](#refcount "引用计数")

size_t uv\_handle_size(uv\_handle_type type)
  返回handle的size,对于FFI绑定很有用,不需要知道hanlde的布局

## 其它杂项函数
 接下来的API函数使用uv\_handle_t做参数,但是只在特定的handle有作用

int **uv\_send_buffer_size**(uv\_handle_t\* handle, int\* value)
  获取或者设置操作系统在socket上使用的发送缓冲去的大小
  如果\*value= =0,返回当前的缓冲区大小,否则使用\*value的值设置新的发送缓冲区大小
  这个函数对UNIX上的TCP,pipe,UDP handle,以及windows上的TCP,UDP有效
  > linux下为设置两倍大小,并且返回初始*value的两倍大小的值

int **uv\_recv_buffer_size**(uv\_handle_t\* *handle*,int \* value)
  获取或者设置操作系统在socket上使用的接收缓冲区的大小
  如果\*value==0,返回当前的缓冲区大小,否则使用\*value的值设置新的缓冲区大小
  这个函数对UNIX上的TCP,pipe,UDP handle,以及windows上的TCP,UDP有效
  > linux下为设置两倍大小,并且返回初始*value的两倍大小的值

int **uv\_fileno**(const uv\_hanlde_t\* *handle*,uv_os_fd_t\* fd)
  获取平台相关的文件描述符
  支持的handle:TCP,pipe,TTY,UDP,poll.传递任何其他的handle失败并且返回错误吗UV\_EINVAL  
  如果handle没有关联文件描述符或者handle已经关闭,函数返回UV_EBADF 
  >警告:使用该函数要特别小心,libuv假定文件描述符在它的控制之下,任何对文件描述符的修改都可能导致故障

## <a name="refcount" id="refcount">引用计数</a>
 libuv事件循环如果运行在default模式下会一直执行直到不存在激活和引用的handle.用户可以对激活的handle解引用来提早结束循环,例如在调用uv\_timer_start()之后调用uv\_unref()

 handle可以被引用或者解引用,引用计数机制并没有使用计数器所以所有操作是幂等的
 默认情况下所有handle一旦激活,就是被引用状态,参见uv\_is_active()

​     


​    