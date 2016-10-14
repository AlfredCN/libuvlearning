TTY 表示控制台的流,uv\_stream\_t的子类

## 数据类型

**`uv_tty_t`**
 tty类型

**`uv_tty_mode_t`**

 1.2.0版本新加
```c
typedef enum {
    UV_TTY_MODE_NORMAL, //初始/普通模式  
    UV_TTY_MODE_RAW, //原始输入模式(windows上ENABLE_WINDOW_INPUT选项也会开启)
    UV_TTY_MODE_IO // 二进制安全I/O模式,只在unix有效
} uv_tty_mode_t;
```

## API

**`int uv_tty_init(uv_loop_t* loop, uv_tty_t* handle, uv_file fd, int readable)`**
指定描述符来初始化一个新的TTY流,
通常:
* 0 =>stdin
* 1 =>stdout
* 2 =>stderr

readable,表明是否准备调用uv_read_start(),stdin可读,stdout不可读

unix平台这个方法会使用[ttyname_r\(3\)](http://linux.die.net/man/3/ttyname_r)确定终端fd的路径,如果是一个tty的话,打开并使用它,这使得libuv可以把tty设置非阻塞模式,而不影响其他共享tty的进程.

在不支持ioctl中 TIOCGPTN 或者 TIOCPTYGNAME选项的系统中,这个方法不是线程安全的,例如OpenBsd,和Solaris.

>如果重开TTY失败,libuv会使用阻塞模式写不可读的TTY流.

1.9.0版本开始tty路径通过[ttyname_r\(3\)](http://linux.die.net/man/3/ttyname_r)决定,之前的版本直接使用/dev/tty.

1.5.0版本修改::在UNIX系统上,通过指向文件的描述符初始化TTY流会返回UV_EINVAL错误 

**`int uv_tty_set_mode(uv_tty_t* handle, uv_tty_mode_t mode)`**

设置终端模式

**`int uv_tty_reset_mode(void)`**

在程序退出的时候调用来重置终端的模式为默认值.以备之后的进程使用

这个方法在UNIX系统上是异步信号安全的,但是在执行uv_tty_set_mod()e的过程中调用的话会失败返回UV_EBUSY的错误码.

**`int uv_tty_get_winsize(uv_tty_t* handle, int* width, int* height)`**

获取终端窗口的大小,成功的话返回0
