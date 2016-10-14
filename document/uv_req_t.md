# uv\_req_t 请求基类
*uv\_req_t*所有request类型的基类
结构体使用字段对齐来使其他request类型可以转换成uv\_req_t.这里定义的所有API函数适用于任何requst类型

## 数据类型
**uv\_req_t**
 libuv 所有request的基础结构

**uv\_any_req**
 所有request类型的union

## 公有成员

void\* **uv\_req_t.data**
   用户自己存放任何数据,libuv不使用
uv\_req_type **un_req_t.type**
   标明request的类型,只读
    ​```c
    typedef enum {
    UV_UNKNOWN_REQ = 0,
    UV_REQ,//请求
    UV_CONNECT,//新建连接
    UV_WRITE,//写数据
    UV_SHUTDOWN,//关闭
    UV_UDP_SEND,//udp-发送
    UV_FS,//文件系统操作
    UV_WORK,//异步任务
    UV_GETADDRINFO,//取地址
    UV_GETNAMEINFO,//取host
    UV_REQ_TYPE_PRIVATE,//...内部用的一些req类型
    UV_REQ_TYPE_MAX,
    } uv_req_type;
    ​```
## API
int **uv_cancel**(uv_req_t\* req)
  取消待定的请求,如果请求正在执行或者执行完毕则失败
  成功返回0,失败错误码
  现在只支持:uv_fs_t,uv_getaddrinfo_t,uv_getnameinfo_t 和uv_work_t
  取消的请求他们的callback函数可能会在之后的时间点被调用,所以在回调调用之前释放request关联的内存是不安全的

  下面是取消操作如何报告给回调函数:
* uv_fs_t 请求在req->result=UV_ECANCLED
* uv_work_t,uv_getaddrinfo_t,uv_getnameinfo_t会使用status==UV_ECANCLED来调用回调函数

size_t **uv_req_size**(uv_req_type type)
  返回给定请求类型的大小  