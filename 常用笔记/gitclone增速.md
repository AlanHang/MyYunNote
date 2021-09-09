只让github使用代理

http代理设置，1081端口修改成自己的本地代理端口

git config --global http.https://github.com.proxy https://127.0.0.1:1081

git config --global https.https://github.com.proxy https://127.0.0.1:1081

\# socks5协议，1080端口修改成自己的本地代理端口

git config --global http.https://github.com.proxy socks5://127.0.0.1:1080

git config --global https.https://github.com.proxy socks5://127.0.0.1:1080

参考网页

https://juejin.im/post/6844903862961176583

