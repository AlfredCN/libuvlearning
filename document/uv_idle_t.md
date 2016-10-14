每次loop循环的时候在uv_prepare_t之前执行

> 和prepare handle的区别是,只要有激活的idlehandle,网络阻塞I/O操作的超时时间会设置为0

## 类型
**uv\_idle_t**
    idle handle类型
void (\* **uv\_idle_cb**)(uv\_idle_t\* handle)
    uv\_idle_start()的callback定义

## API

int uv\_idle_init(uv\_loop_t\* loop,uv\_idle_t\* idle)
    handle初始化

int uv\_idle_start(uv\_idle_t\* idle,uv\_idle_cb cb)
    开始

int uv\_idle_stop(uv\_idle_t* idle)
    结束
 


