文件系统事件handle,允许用户监听给定路径的改变.比如文件重命名或者其他变化.这个handle使用各个系统上提供的接口来实现

>对于AIX系统,需要安装非默认的IBM bos.ahafs 包, AIX 文件系统事件有一些限制


>    * ahafs的事件监测跟进程关联并且不是线程安全的.因此同一事件的每个监测都要生成一个单独的进程 ==??


>    * 如果只监听包含文件的文件夹,文件改变的事件是获取不到的

## 数据类型

**`uv_fs_event_t`**
  fs时间 handle类型

**`void (*uv_fs_event_cb)(uv_fs_event_t* handle, const char* filename, int events, int status)`**
uv_fs_event_start()方法的回调,会在handle开启之后重复被调用.如果handle是通过目录开启的,filename参数是相对于开启目录的相对路径,events参数是要监控的事件, uv_fs_event枚举的或操作

**`uv_fs_event`**
文件系统事件类型
```c
enum uv_fs_event {
    UV_RENAME = 1, //重命名
    UV_CHANGE = 2 // 改变
};
```
**`uv_fs_event_flags`**
传递给uv_fs_event_start()的标记,来控制监控行为.
```c
enum uv_fs_event_flags {
    /*
    * 默认是给定一个目录名,监听目录下的所有事件. 这个选项表明只监听目录本身的事件,并不关心目录下的每个文件,**目前尚未实现**
    */
    UV_FS_EVENT_WATCH_ENTRY = 1,
    /*
    * 默认uv_fs_event使用内核接口inotify/或者看queue监控事件,这在远程文件系统例如NFS格式上是不能工作的. 这个选项回退到调用周期间隔调用stat()的方式检测事件变化.**目前尚未实现**
    */
    UV_FS_EVENT_STAT = 2,
    /*
    * 默认不监听子目录的事件.
    * 这个选项监听每一个子目录.
    */
    UV_FS_EVENT_RECURSIVE = 4
};
```
## API
**`int uv_fs_event_init(uv_loop_t* loop, uv_fs_event_t* handle)`**

初始化

**`int uv_fs_event_start(uv_fs_event_t* handle, uv_fs_event_cb cb, const char* path, unsigned int flags)`**

 开启监听某个目录
flags目前只在windows和osx上支持UV_FS_EVENT_RECURSIVE

**`int uv_fs_event_stop(uv_fs_event_t* handle)`**
停止监听

**`int uv_fs_event_getpath(uv_fs_event_t* handle, char* buffer, size_t* size)`**

获取handle监听的目录.buffer内存必须预分配. 
0成功,<0失败,成功的时候buffer包含路径,size包含路径长度.buffer长度不够的时候返回UV_ENOBUFS错误,size设置成需要的长度.
1.3.0版本开始: 返回的长度不包括结尾null字符,buffer也不是加结尾null.
