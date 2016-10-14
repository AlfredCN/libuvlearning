libuv提供了各种跨平台的同步/异步文件系统操作.文档中的所有方法都接受一个callback,可以被设置为NULL.如果是NULL的话同步完成,否则异步执行

所有文件操作在线程池上运行.查看[线程池工作调度]()查看有关线程池大小的信息.

##数据类型

**`uv_fs_t`**

 文件系统请求类型.

**`uv_timespec_t`**

等同于struct timespec
```c
typedef struct {
    long tv_sec; //秒
    long tv_nsec;//纳秒
} uv_timespec_t;
```

**`uv_stat_t`**

等同于struct stat
```c
typedef struct {
    uint64_t st_dev;
    uint64_t st_mode;
    uint64_t st_nlink;
    uint64_t st_uid;
    uint64_t st_gid;
    uint64_t st_rdev;
    uint64_t st_ino;
    uint64_t st_size;
    uint64_t st_blksize;
    uint64_t st_blocks;
    uint64_t st_flags;
    uint64_t st_gen;
    uv_timespec_t st_atim;
    uv_timespec_t st_mtim;
    uv_timespec_t st_ctim;
    uv_timespec_t st_birthtim;
} uv_stat_t;
```

**`uv_fs_type`**
文件系统请求类型

```c
typedef enum {
    UV_FS_UNKNOWN = -1,
    UV_FS_CUSTOM,
    UV_FS_OPEN,  //打开
    UV_FS_CLOSE, //关闭
    UV_FS_READ,  //读取
    UV_FS_WRITE, //写
    UV_FS_SENDFILE, //发送?
    UV_FS_STAT,    //获取状态
    UV_FS_LSTAT,   //
    UV_FS_FSTAT,   // 
    UV_FS_FTRUNCATE, // 截断??
    UV_FS_UTIME,   //
    UV_FS_FUTIME,  // 
    UV_FS_ACCESS,  // 
    UV_FS_CHMOD,
    UV_FS_FCHMOD,
    UV_FS_FSYNC,
    UV_FS_FDATASYNC,
    UV_FS_UNLINK,
    UV_FS_RMDIR,
    UV_FS_MKDIR,
    UV_FS_MKDTEMP,
    UV_FS_RENAME,
    UV_FS_SCANDIR,
    UV_FS_LINK,
    UV_FS_SYMLINK,
    UV_FS_READLINK,
    UV_FS_CHOWN,
    UV_FS_FCHOWN,
    UV_FS_REALPATH
} uv_fs_type;
```

**`uv_dirent_t`**

跨平台的目录属性,等同 struct dirent,用于 uv\_fs\_scandir\_next()

```c
//类型
typedef enum {
    UV_DIRENT_UNKNOWN,
    UV_DIRENT_FILE,
    UV_DIRENT_DIR,
    UV_DIRENT_LINK,
    UV_DIRENT_FIFO,
    UV_DIRENT_SOCKET,
    UV_DIRENT_CHAR,
    UV_DIRENT_BLOCK
} uv_dirent_type_t;

typedef struct uv_dirent_s {
    const char* name;
    uv_dirent_type_t type;
} uv_dirent_t;
```
## 公有成员

**`uv_loop_t* uv_fs_t.loop`**
开启文件请求的loop,完成的回调也会在这个loop上执行. 只读.

**`uv_fs_type uv_fs_t.fs_type`**

请求类型

**`const char* uv_fs_t.path`**

请求关联的目录

**`ssize_t uv_fs_t.result`**
请求的结果,< 0 表示出错,否则成功. 在uv_fs_read() 或者 uv_fs_write()的请求上表示读/写的字节数.

**`uv_stat_t uv_fs_t.statbuf`**
保存uv_fs_stat()的请求结果还有其他stat请求的结果.

**`void\* uv_fs_t.ptr`**
uv_fs_readlink()的结果,作为statbuf的别名.

## API

**`void uv_fs_req_cleanup(uv_fs_t* req)`**

清理请求. 必须在请求完成的时候调用来收回libuv分配的内存.

**`int uv_fs_close(uv_loop_t* loop, uv_fs_t* req, uv_file file, uv_fs_cb cb)`**

关闭文件,和[close\(2\)](http://linux.die.net/man/2/close)相同.

**`int uv_fs_open(uv_loop_t* loop, uv_fs_t* req, const char* path, int flags, int mode, uv_fs_cb cb)`**

打开文件,和[open(2)](http://linux.die.net/man/2/open).相同

> windows平台libuv使用CreateFileW打开文件,文件总是使用二进制模式打开. 所以不支持O_BINARY和O_TEXT选项.

**`int uv_fs_read(uv_loop_t* loop, uv_fs_t* req, uv_file file, const uv_buf_t bufs[], unsigned int nbufs, int64_t offset, uv_fs_cb cb)`**

读文件,和[preadv(2)](http://linux.die.net/man/2/preadv).相同

**`int uv_fs_unlink(uv_loop_t* loop, uv_fs_t* req, const char* path, uv_fs_cb cb)`**

解除文件引用,和[unlink(2)](http://linux.die.net/man/2/unlink).相同

**`int uv_fs_write(uv_loop_t* loop, uv_fs_t* req, uv_file file, const uv_buf_t bufs[], unsigned int nbufs, int64_t offset, uv_fs_cb cb)`**

写文件,和[pwritev(2)](http://linux.die.net/man/2/pwritev).相同

**`int uv_fs_mkdir(uv_loop_t* loop, uv_fs_t* req, const char* path, int mode, uv_fs_cb cb)`**

创建目录,和[mkdir(2)](http://linux.die.net/man/2/mkdir).相同

目前mode在Windows平台上不支持.

**`int uv_fs_mkdtemp(uv_loop_t* loop, uv_fs_t* req, const char* tpl, uv_fs_cb cb)`**

创建临时文件夹,和[mkdtemp(3)](http://linux.die.net/man/3/mkdtemp).相同

>返回值保存在req->path中.

**`int uv_fs_rmdir(uv_loop_t* loop, uv_fs_t* req, const char* path, uv_fs_cb cb)`**

删除目录 和[rmdir(2)](http://linux.die.net/man/2/rmdir).相同

**`int uv_fs_scandir(uv_loop_t* loop, uv_fs_t* req, const char* path, int flags, uv_fs_cb cb)`**

**`int uv_fs_scandir_next(uv_fs_t* req, uv_dirent_t* ent)`**

扫描目录,和[scandir(3)](http://linux.die.net/man/3/scandir)等同,只是API稍微有些改变. 一旦uv_fs_cb的回调被调用,用户就可以调用uv_fs_scandir_next()获取目录下一个子节点,信息保存在ent中. 没有子节点的时候返回UV_EOF.

和scandir(3)不同的是,这个方法不返回"."和"..".

>在linux上获取节点类型只有特定的几个文件系统(btrfs, ext2, ext3 and ext4 at the time of this writing) ,查看[getdents(2)](http://linux.die.net/man/2/getdents)帮助页面.

**`int uv_fs_stat(uv_loop_t* loop, uv_fs_t* req, const char* path, uv_fs_cb cb)`**

**`int uv_fs_fstat(uv_loop_t* loop, uv_fs_t* req, uv_file file, uv_fs_cb cb)`**

**`int uv_fs_lstat(uv_loop_t* loop, uv_fs_t* req, const char* path, uv_fs_cb cb)`**

分别等同于[stat(2)](http://linux.die.net/man/2/stat)stat(2), [fstat(2)](http://linux.die.net/man/2/fstat)fstat(2) 和[lstat(2)](http://linux.die.net/man/2/lstat).

**`int uv_fs_rename(uv_loop_t* loop, uv_fs_t* req, const char* path, const char* new_path, uv_fs_cb cb)`**
等同于[rename(2)]((http://linux.die.net/man/2/rename)).

**`int uv_fs_fsync(uv_loop_t* loop, uv_fs_t* req, uv_file file, uv_fs_cb cb)`**
等同于[fsync(2)]((http://linux.die.net/man/2/fsync)).

**`int uv_fs_fdatasync(uv_loop_t* loop, uv_fs_t* req, uv_file file, uv_fs_cb cb)`**
等同于[fdatasync(2)](http://linux.die.net/man/2/fdatasync).

**`int uv_fs_ftruncate(uv_loop_t* loop, uv_fs_t* req, uv_file file, int64_t offset, uv_fs_cb cb)`**
等同于[ftruncate(2)](http://linux.die.net/man/2/ftruncate).

int uv_fs_sendfile(uv_loop_t* loop, uv_fs_t* req, uv_file out_fd, uv_file in_fd, int64_t in_offset, size_t length, uv_fs_cb cb)

等同于[sendfile(2)](http://linux.die.net/man/2/sendfile).

int uv_fs_access(uv_loop_t* loop, uv_fs_t* req, const char* path, int mode, uv_fs_cb cb)

在unix平台上等同于[access(2)](http://linux.die.net/man/2/access). windows 平台使用GetFileAttributesW().

int uv_fs_chmod(uv_loop_t* loop, uv_fs_t* req, const char* path, int mode, uv_fs_cb cb)
int uv_fs_fchmod(uv_loop_t* loop, uv_fs_t* req, uv_file file, int mode, uv_fs_cb cb)

分别等同于 [chmod(2)](http://linux.die.net/man/2/chmod)和[fchmod(2)](http://linux.die.net/man/2/fchmod).

int uv_fs_utime(uv_loop_t* loop, uv_fs_t* req, const char* path, double atime, double mtime, uv_fs_cb cb)
int uv_fs_futime(uv_loop_t* loop, uv_fs_t* req, uv_file file, double atime, double mtime, uv_fs_cb cb)

分别等同于 [utime(2)](http://linux.die.net/man/2/utime)和[futime(2)](http://linux.die.net/man/2/futime).

>注AIX:只支持AIX 7.1或更高的版本.低版本调用返回UV_ENOSYS错误.
>1.10.0开始:windows平台支持亚秒级的精度

int uv_fs_link(uv_loop_t* loop, uv_fs_t* req, const char* path, const char* new_path, uv_fs_cb cb)

等同于[link(2)](http://linux.die.net/man/2/link).

int uv_fs_symlink(uv_loop_t* loop, uv_fs_t* req, const char* path, const char* new_path, int flags, uv_fs_cb cb)

等同于[symlink(2)](http://linux.die.net/man/2/symlink).

>Windows平台,flags可以用来控制怎么创建符号链接:
>UV_FS_SYMLINK_DIR: 表明path指向一个路路径.
>UV_FS_SYMLINK_JUNCTION:使用junction创建符号链接.

int uv_fs_readlink(uv_loop_t* loop, uv_fs_t* req, const char* path, uv_fs_cb cb)

等同于[readlink(2)](http://linux.die.net/man/2/readlink).

int uv_fs_realpath(uv_loop_t* loop, uv_fs_t* req, const char* path, uv_fs_cb cb)

unix上等同于[realpath(2)](http://linux.die.net/man/2/realpath).windows上使用GetFinalPathNameByHandle().

> 警告,node使用的时候发现,这个方法在一些平台上有特定的警告.


>macOS 和其他BSDs: 在解析路径的时候出现超过32个符号链接的时候会失败返回UV_ELOOP错误,这个限制是硬编码在代码中的没办法回避.
>Windows: 有一些例外的情况会失败:
    * 通过工具(eg ImDisk)创建的闪存盘上的目录没办法解析
    * 驱动盘符的大小写不一致的时候
    * SUBST创建的盘符中的路径

>注意 这个方法在winxp和win2003上没有实现,在这些系统上返回UV_ENOSYS

1.8.0版本新加

int uv_fs_chown(uv_loop_t* loop, uv_fs_t* req, const char* path, uv_uid_t uid, uv_gid_t gid, uv_fs_cb cb)
int uv_fs_fchown(uv_loop_t* loop, uv_fs_t* req, uv_file file, uv_uid_t uid, uv_gid_t gid, uv_fs_cb cb)

分别等同于chown(2) 和 fchown(2), windows平台上没有实现