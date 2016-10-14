# uv_timer_t 定时器handle
处理定时操作

## 数据类型
**uv_timer_t**
  定时器handle
void **(\*uv\_timer_cb)**(uv\_timer_t\* *handle*)
  uv\_timer_start()的回调定义

## 公有成员
N/A
> 参见:uv\_hanle_t的公有成员

## API

int **uv\_timer_init**(uv\_loop_t\* loop,uv\_timer_t\* handle)
  handle初始化  
int **uv\_timer_start**(uv\_timer_t\* handle,uv\_timer_cb cb, uint64_t timeout,uin64_t repeat)
  开始计时器,timeout和repeat单位毫秒
  如果timeout==0,回调下次循环迭代的时候调用,
  如果repeat~=0,在timeout毫秒时间后调用,然后在repeat指定的时间后重复
int **uv_timer_stop**(uv_timer_t\* *handle* )
  停止定时器,回调不会再调用
int **uv_timer_again**(uv_timer_t\* *handle*)
  停止计时器,如果计时器是重复的在repeat值指定的时间之后再重启,如果timer没有start过返回UV_EINVAL
void **uv_timer_set_repeat**(uv_timer_t\* handle,uint64_t repeat)
  设置循环周期(毫秒)
  > 如果repeat值是在callback回调中设置的话,不会立刻起效果.如果定时器之前是不重复的,会先停止.如果之前是重复的,下次迭代会使用原来的repeat值

uint64_t **uv_timer_get_repeat**(const uv_timer_t\* *handle*)
  获取repeat值


 >另: [uv_handle_t](#)中的api接口同样适用