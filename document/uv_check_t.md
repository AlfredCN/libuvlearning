# uv\_check_t Check handle
每次循环迭代中,在I/O轮询之后调用回调

## 数据类型
**uv\_check_t**
  check 类型
void **(*uv\_check_cb)(uv_check_t\* *handle*)
  uv\_check_start()的回调定义

## 公有成员
N/A
> 另:查看 **uv\_handle_t**的公有成员

## API
int **uv\_check_init**(uv\_loop_t\* loop,uv\_check_t* check)
  初始化 
int **uv\_cehck_start**(uv\_check_t\* check, uv\_check_cb cb)
  开始
int **uv\_check_stop**(uv\_check_t* check)
  结束

>另:查看**uv\_handle_t**的API函数