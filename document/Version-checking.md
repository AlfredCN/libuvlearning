从1.0.0版本开始,libuv遵循[语义版本](http://semver.org/lang/zh-CN/)(emeantic versioning)的设计.这意味着新的api是在一个大版本更新的时候引入.在这一节中将可以看到用来做版本检测的所有的宏和方法,以使代码可以兼容多个libuv版本.

# 宏

** UV_VERSION_MAJOR ** 大版本号

** UV_VERSION_MINOR ** 小版本号

** UV_VERSION_PATCH ** patch号

** UV_VERSION_IS_RELEASE ** 1表示release版本,0表示开发版本

** UV_VERSION_SUFFIX ** 版本后缀,例如"rc"

** UV_VERSION_HEX ** 把版本号打包入一个整形中返回,每个部分用八位表示.

# 方法

unsigned int *uv_version*(void)
返回UV_VERSION_HEX

const char* uv_version_string(void)
返回版本字符串,对于开发版本包含版本后缀




 

