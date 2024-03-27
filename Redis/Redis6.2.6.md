[TOC]

# Redis 单机安装

redis官方地址`https://redis.io/`

1. 安装Redis依赖

   ```bash 
   yum install -y gcc tcl
   ```

2. 上传安装包并解压

3. 编译redis

   ```bash
    make && make install
   ```

4. 检查是否安装成功
![image-20240326143229820](Redis6.2.6.assets/image-20240326143229820.png)
   
5. redis默认启动 `redis-server`

6. 指定配置启动(redis解压后的文件)
   ![image-20240326143543971](Redis6.2.6.assets/image-20240326143543971.png)

7. redis中的配置

   ![image-20240326144408469](Redis6.2.6.assets/image-20240326144408469.png)
   
   ![image-20240326144630707](Redis6.2.6.assets/image-20240326144630707.png)
   
8. 启动

   ![image-20240326145502913](Redis6.2.6.assets/image-20240326145502913.png)

9. 开机自启动

   ```bash
   vi /etc/systemd/system/redis.service
   
   --- 
   [unit]
   Description=redis-server
   After=network.target
   
   
   [Service]
   Type=forking
   ExecStart=/usr/local/bin/redis-server /mnt/disk1/redis_6.2.6/redis-6.2.6/redis.conf
   PrivateTmp=true
   
   
   [Install]
   WantedBy=multi-user.target
   
   ---
   systemctl daemon-reload
   systemctl start redis
   systemctl enable redis
   ```

# Redis 客户端

![image-20240326150126311](Redis6.2.6.assets/image-20240326150126311.png)

# Redis 数据结构

![image-20240326152913457](Redis6.2.6.assets/image-20240326152913457.png)

## Redis通用命令

![image-20240326153635587](Redis6.2.6.assets/image-20240326153635587.png)

## String类型

![image-20240326160223906](Redis6.2.6.assets/image-20240326160223906.png)

![image-20240326160252088](Redis6.2.6.assets/image-20240326160252088.png)

![image-20240326160530250](Redis6.2.6.assets/image-20240326160530250.png)

## Hash类型

![image-20240326161155376](Redis6.2.6.assets/image-20240326161155376.png)

![image-20240326161224806](Redis6.2.6.assets/image-20240326161224806.png)

## List类型

![image-20240326162332736](Redis6.2.6.assets/image-20240326162332736.png)

![image-20240326162412447](Redis6.2.6.assets/image-20240326162412447.png)

## Set类型

![image-20240326162545312](Redis6.2.6.assets/image-20240326162545312.png)

![image-20240326162625408](Redis6.2.6.assets/image-20240326162625408.png)

![image-20240326162653810](Redis6.2.6.assets/image-20240326162653810.png)

![image-20240326162712880](Redis6.2.6.assets/image-20240326162712880.png)

# Redis Java客户端

![image-20240326163210265](Redis6.2.6.assets/image-20240326163210265.png)

## Jedis

![image-20240326163356364](Redis6.2.6.assets/image-20240326163356364.png)

![image-20240326163451805](Redis6.2.6.assets/image-20240326163451805.png)

![image-20240326163636951](Redis6.2.6.assets/image-20240326163636951.png)

## SpringDataRedis

![image-20240326163845203](Redis6.2.6.assets/image-20240326163845203.png)

![image-20240326164108897](Redis6.2.6.assets/image-20240326164108897.png)![image-20240326164231282](Redis6.2.6.assets/image-20240326164231282.png)

![image-20240326165807944](Redis6.2.6.assets/image-20240326165807944.png)

![image-20240326165838893](Redis6.2.6.assets/image-20240326165838893.png)

![image-20240326170348217](Redis6.2.6.assets/image-20240326170348217.png)

![image-20240326170558540](Redis6.2.6.assets/image-20240326170558540.png)

![image-20240326170906864](Redis6.2.6.assets/image-20240326170906864.png)

![image-20240326171016908](Redis6.2.6.assets/image-20240326171016908.png)

![image-20240326171050306](Redis6.2.6.assets/image-20240326171050306.png)

![image-20240326171253475](Redis6.2.6.assets/image-20240326171253475.png)





