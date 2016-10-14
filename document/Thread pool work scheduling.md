libuv提供线程池,用来执行用户代码,以及在loop线程上获取通知. 在libuv内部线程池用来执行所有的文件系统操作以及getaddrinfo和getnameinfo请求.

默认线程池大小是4,但可以在启动的时候这设置环境变量UV_THREADPOOL_SIZE来修改(最大值128).

线程池是全局的,并且在所有事件循环中共享. 当特定的方法(eg:uv_queue_work())使用线程池的时候,libuv 预分配并且初始化UV_THREADPOOL_SIZE指定的最大数量的线程.这会带来相对较小的内存消耗(128个线程大约1M)并且增加的线程的性能.

>注意尽管线程池是全局并且在全部事件循环中共享,方法并不是线程安全的.

## 数据类型

**`uv_work_t`**

Work请求类型

**`void (*uv_work_cb)(uv_work_t* req)`**

uv_queue_work()的回调在线程池上执行.

**`void (*uv_after_work_cb)(uv_work_t* req, int status)`**

传递到uv_queue_work()的回调,任务执行完毕之后,在loop的线程上执行.如果任务通过uv_cancel被取消,status的值为UV_ECANCELED.

## 公有成员

**`uv_loop_t* uv_work_t.loop`**
执行任务的loop,任务执行完之后通知回调会在该loop的线程上执行. 只读.


## API

**`int uv_queue_work(uv_loop_t* loop, uv_work_t* req, uv_work_cb work_cb, uv_after_work_cb after_work_cb)`**

初始化一个异步任务,将会在线程池上执行work_cb并且等任务执行完之后再loop的线程上执行after_work_cb回调.

请求可以使用uv_cancel()取消.