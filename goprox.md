# go设置代理



## windows

#### 临时方法，打开终端，分辨敲入如下命令：

~~~bash
set http_proxy=http://192.168.36.178:1024

set https_proxy=%http_proxy%
~~~



如果是ss或者ssr把http改成socks5 地址改成127.0.0.1:1024

#### 永久，更改环境变量,新增系统环境变量http_proxy和https_proxy

~~~bash
http_proxy socks5://127.0.0.1:1080  # 代理地址
https_proxy socks5://127.0.0.1:1080
~~~





## linux 书写shell脚本



```shell
#!/bin/bash
export http_proxy=socks5://127.0.0.1:1080 # 代理地址
export https_proxy=$http_proxy
export | grep http
```

添加可执行权限

~~~bash
chmod +x proxy.sh
~~~

生效

~~~bash
source proxy.sh
~~~

取消

~~~bash
unset http_proxy
~~~

 