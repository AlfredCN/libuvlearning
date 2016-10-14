process handle创建新的子进程,允许用户控制它并且通过流(streams)来和子进程通信.

## 数据类型

** `uv_process_t` **
process handle类型

** `uv_process_options_t` **
创建子进程的选项,传递给uv_spawn()方法
```c
typedef struct uv_process_options_s {
    uv_exit_cb exit_cb; //子进程退出的回调
    const char* file;   //子进程的执行文件路径
    char** args;        //子进程的命令行参数,args[0]必须是子进程的执行文件路径,windows环境使用的createProcess 会将参数列表链接成字符串,导致奇怪的错误,使用UV_PROCESS_WINDOWS_VERBATIM_ARGUMENTS 选项来避免出错
    char** env;         //环境变量,传递NULL的话使用父进程的值
    const char* cwd;    // 当前执行路径
    unsigned int flags; // 选项用来控制uv_spwan()的行为
    int stdio_count;    // stdio个数
    uv_stdio_container_t* stdio; // 传递给子进程的stdio列表,惯例stdio[0]指向stdid,fd1 指向stdout,fd 2 指向stderr
    uv_uid_t uid;       //子进程的用户id
    uv_gid_t gid;       // 子进程的组id
} uv_process_options_t;
```

`void (*uv_exit_cb)(uv_process_t*, int64_t exit_status, int term_signal)`

子进程退出的回调,exit_status表示退出状态,term_signal如果有值的话表示导致进程退出的信号

`uv_process_flags`
设置field字段

```c
enum uv_process_flags {
    UV_PROCESS_SETUID = (1 << 0), // 子进程用户id
    UV_PROCESS_SETGID = (1 << 1), // 子进程组id
    /*
    * 该选项只在windows平台有效,unix无效
    * 在转换参数列表的时候,不要把参数包裹在引号中或者做其他的转义
    */
    UV_PROCESS_WINDOWS_VERBATIM_ARGUMENTS = (1 << 2),
    
    /*
    * 设置子进程为脱离状态,表明子进程是进程组的leader,在父进程退出的时候也可以继续运行
    * 子进程依然会是父进程的事件循环处于有效状态,除非父进程在子进程的handle上调用uv_unref()
    */
    UV_PROCESS_DETACHED = (1 << 3),
   
   /*
    * 隐藏子进程的控制台窗口,只在windows平台有效
    */
    UV_PROCESS_WINDOWS_HIDE = (1 << 4)
};
```

`uv_stdio_container_t`
保存传递给子进程的每个fd以及stream handle的容器

```c
typedef struct uv_stdio_container_s {
    uv_stdio_flags flags; // stdio容器怎么传递给子进程的标记
    union {
        uv_stream_t* stream;
        int fd;
    } data;               // stream或者fd
} uv_stdio_container_t;
```

`uv_stdio_flags`
设置stdio怎么传递给子进程

```c
typedef enum {
    UV_IGNORE = 0x00,  
    UV_CREATE_PIPE = 0x01,
    UV_INHERIT_FD = 0x02,
    UV_INHERIT_STREAM = 0x04,
    /*
    * 指定UV_CREATE_PIPE选项的时候, UV_READABLE_PIPE and UV_WRITABLE_PIPE
    * 以子进程的角度表明数据流的方向,两个都指定的话表明创建双向的数据流
    */
    UV_READABLE_PIPE = 0x10,
    UV_WRITABLE_PIPE = 0x20
} uv_stdio_flags;
```

## 公有成员

**`uv_process_t.pid` **
子进程的pid,在调用uv_spwan()之后设置

## API

** `void uv_disable_stdio_inheritance(void)` ** 

禁止文件描述符继承,好处是子进程不会意外的继承这些文件描述符,建议在程序中在继承的文件描述符被关闭或者复制之前,尽可能早的调用该方法.

>这个方法会尽最大努力防止继承,但是不能保证可以检测到所有继承的文件描述符,在windows平台上效果要好于windows平台

** `int uv_spawn(uv_loop_t* loop, uv_process_t* handle, const uv_process_options_t* options)`**

初始化process handle并且开启子进程,如果创建成功返回0否则返回负值的错误码

有可能的错误包括:要执行的程序不存在,没权限设置用户/组id,没有足够的内存分配子进程

** `int uv_process_kill(uv_process_t* handle, int signum)` **
发送特定的信号给子进程

** `int uv_kill(int pid, int signum)` **
发送特定的信号给pid指定的进程









