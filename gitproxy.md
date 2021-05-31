# Git设置代理解析git clone慢问题



## 设置shadowsocks代理

~~~bash
 git config --global http.proxy socks5://127.0.0.1:1080

 git config --global https.proxy socks5://127.0.0.1:1080
~~~



## 设置http代理

~~~bash
git config --global https.proxy http://127.0.0.1:1080 

git config --global https.proxy https://127.0.0.1:1080 
~~~



## 取消代理

~~~bash
git config --global --unset http.proxy 

git config --global --unset https.proxy
~~~

