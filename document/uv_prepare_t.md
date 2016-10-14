# uv\_prepare_t Prepare handle
每次循环迭代中,在I/O轮询操作之前都会调用指定的回调

## 数据类型
**uv\_prepare_t**
  prepare handle
void **(\*uv\_prepare_cb)**(uv\_prepare_t\* *handle*)
  uv\_prepare_start()的回调

## 公有成员 
N/A
> 另:查看uv_handle_t中公有成员

## API
int **uv\_prepare_init**(uv\_loop_t\* loop,uv\_prepare_t\* prepare)
  初始化
int **uv\_prepare_start**(uv\_prepare_t\* prepare,uv\_prepare_cb cb)
  开始
int **uv\_prepare_stop**(uv\_prepare_t* prepare)
  停止
> 另:查看uv\_handle_t中API函数