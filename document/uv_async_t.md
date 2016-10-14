async handle允许用户"唤醒"事件循环,可以在其他的线程触发callback的调用,callback依然在loop所在的线程执行

## 数据类型
```c
uv_async_t
 async handle类型

void (* uv_async_cb )(uv_async_t* handle)
  uv_async_init()函数的callback定义
```
## API

`int uv_async_init(uv_loop_t* loop, uv_async_t* async, uv_async_cb async_cb)`

初始化handle,callback可以传NULL,handle会立即被启动

`int uv_async_send(uv_async_t* async)` 

唤醒loop.并且调用回调函数,在任意线程上调用都是安全的,callback函数会在loop线程上调用.

>libuv将会合并uv\_async\_send()的调用,如果uv\_async\_send()在callback之前调用5次的话callback函数只会执行一次


