# git使用代理

对于ssh方式的git没有效果

```shell
# 只让github使用代理 http代理设置，1081端口修改成自己的本地代理端口

git config --global http.https://github.com.proxy https://127.0.0.1:1081

git config --global https.https://github.com.proxy https://127.0.0.1:1081

# socks5协议，1080端口修改成自己的本地代理端口

git config --global http.https://github.com.proxy socks5://127.0.0.1:1080

git config --global https.https://github.com.proxy socks5://127.0.0.1:1080

# 使用全局代理

git config --global http.proxy https://127.0.0.1:1081

git config --global https.proxy https://127.0.0.1:1081
```

# 解决ssh: connect to host github.com port 22: Connection refused

1. 执行`ssh git@github.com`命令得到报错信息

   `connect to host github.com port 22: Connection refused`

2. 进入.ssh( `cd ~/.ssh` )的配置目录查看，发现ssh目录里少了配置文件config。

3. ```shell
   touch config
   
   # 写入你github的配置信息。（xxxxx@xx.com是你github的注册邮箱）
   Host github.com  
   User xxxxx@xx.com  
   Hostname ssh.github.com  
   PreferredAuthentications publickey  
   IdentityFile ~/.ssh/id_rsa  
   Port 443
   ```

   

