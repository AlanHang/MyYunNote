[toc]

# 获取并修改cookie值

```nginx
  # 配置在location中，以JSESSIONID_TOKEN为例
  set $JSESSIONID_TOKEN "";
  if ($http_cookie ~* "JSESSIONID=([\w-]+?)(?=;|$)") {
      set $JSESSIONID_TOKEN "$1";
    }
  add_header    Set-Cookie  "JSESSIONID=$JSESSIONID_TOKEN;PATH=/" ;
```

# location中使用通配符会自动解析特殊符号和中文

```nginx
  #如下所示，$2为中文时向下传递的也是中文
  location ~ ^/app-(\d+)/([^/]+/(flowIcon|formIcon)/.*$) {
    proxy_pass http://gap-gateway:8080/netoweb/$2?$args;
    proxy_read_timeout 6000;
    proxy_set_header Host             $http_host;
    proxy_set_header origin-domain    app-$1.com;
    proxy_set_header X-Real-IP        $remote_addr;
    proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
  }
  
  #应该改为
    location ~ ^/app-(\d+)/([^/]+/(flowIcon|formIcon)/.*$) {
    if ($request_uri ~ ^/app-(\d+)/([^/]+/(flowIcon|formIcon)/.*$) {
        proxy_pass http://gap-gateway:8080/$2;
    }
    proxy_read_timeout 6000;
    proxy_set_header Host             $http_host;
    proxy_set_header origin-domain    app-$1.com;
    proxy_set_header X-Real-IP        $remote_addr;
    proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
  }
```

# 配置https

```nginx
server {
        listen       35233 ssl;
        server_name  localhost;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH::AES128-SHA:DES-CBC3-SHA:ECC-SM4-CBC-SM3:ECC-SM4-GCM-SM3:ECDHE-SM2-WITH-SMS4-GCM-SM3:ECDHE-SM2-WITH-SMS4-SHA256:ECDHE-SM2-WITH-SMS4-SM3:SM2-WITH-SMS4-SM3:SM2DHE-WITH-SMS4-SM3!NULL:!aNULL:!MD5:!ADH:!RC4:!3DES;
        ssl_verify_client off;
       #ssl_certificate /usr/share/nginx/web/gmkey/server.crt;
       #ssl_certificate_key /usr/share/nginx/web/gmkey/server.key;

       ssl_certificate /usr/share/nginx/web/gmkey/certificate.crt;
       ssl_certificate_key /usr/share/nginx/web/gmkey/private2.key;

       ssl_certificate /usr/share/nginx/web/gmkey/218.60.153.156.cer.pem;
       ssl_certificate_key /usr/share/nginx/web/gmkey/private.key;

       ssl_certificate /usr/share/nginx/web/gmkey/218.60.153.156.cer.pem;
       ssl_certificate_key /usr/share/nginx/web/gmkey/private.key;

        location / {
          proxy_pass http://gaf-minio.eonamsxc-15:9000;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Real-Port $remote_port;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_set_header Host             $http_host;
          proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
  }
```

