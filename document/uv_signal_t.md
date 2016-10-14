在每个事件循环中实现unix形式的信号处理

windows平台上有几个信号可以被模拟

* SIGINT在用户按CTRL+C的时候投递,和unix一样在终端的原始模式(raw mode)启用的时候不触发
* SIGBREAK在用户按CTRL+BREAK的时候投递
* SIGHUP在用户关闭控制台窗口的时候触发,在SIGHUP信号处理的时候有大约10秒左右的时间清理资源,之后会被无条件的关闭.
* SIGWINCH在libuv检测到窗口大小改变的时候触发,在使用uv_tty_t写控制台的时候,SIGWINCH被libuv模拟触发.SIGWINCH不会一直持续检测; libuv只在鼠标指针移动的时候做检测,在原始模式使用可读的uv_tty_t的时候,重设置console的缓冲也会触发.

除了以下信号之外的信号也可悲成功的检测到:SIGILL,SIGABRT,SIGFPE,SIGTERM,SIGKILL,手动调用raise()或者abort()触发的信号也不会被libuv检测到.

> linux上信号SIGRT0和SIGRT1是被NPTL pthread库用来管理线程的,监听这两个事件可能会导致未定义的行为出现,所以强烈建议不要监听这两个信号,后续版本可能会直接拒绝这两个信号

## 数据类型

**`uv_signal_t`**

Signal类型

** `void (*uv_signal_cb)(uv_signal_t* handle, int signum)` **
uv_signal_start()的callback定义.

## 公有成员

** `int uv_signal_t.signum` **
 监控的信号,只读

## API

** `int uv_signal_init(uv_loop_t* loop, uv_signal_t* signal)` **

初始化

**`int uv_signal_start(uv_signal_t* signal, uv_signal_cb cb, int signum)`**

激活signal监控指定的信号

** `int uv_signal_stop(uv_signal_t* signal)`**

停止

