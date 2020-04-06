[文档](https://letsencrypt.org/zh-cn/how-it-works/)
https://www.jianshu.com/p/c5c9d071e395

在 VPS 建站下手动安装 SSL 证书时，如果出现此错误提示：
```
Problem binding to port 80: Could not bind to IPv4 or IPv6.
```
则原因是 nginx 占用了80端口，输入 service nginx stop。然后再次执行证书安装命令，即可顺利安装。

安装完毕后，输入 service nginx start，重启 nginx 服务。


阿里云域名快速安装
https://www.laozuo.org/11729.html
https://www.cmsky.com/lets-encrypt-wildcard-ssl/



# nginx 配置https
直接运行脚本会有提示"It is recommended to install socat first"等错误信息，需要提前安装必备软件包。
## 安装必要的软件包
1. Debian/Ubuntu
```
apt-get update && apt-get install curl -y && apt-get install cron -y && apt-get install socat -y
```
2. CentOS
```
yum update && yum install curl -y && yum install cron -y && yum install socat -y
```
## 安装acme.sh
```
curl https://get.acme.sh | sh
```

## 获取阿里云的`Access Key ID` `Access Key Secret`
1. 申请地址
```
https://ak-console.aliyun.com/#/accesskey
```

2. 将对应的ID 和secret放入系统变量中
在ssh里执行下面的
```
export Ali_Key="对应Access Key ID"
export Ali_Secret="对应Access Key Secret"
```

## 快速签发证书

1. 用默认的目录
```
~/.acme.sh/acme.sh --issue -d javafroum.cn -d *.javafroum.cn --dns dns_ali
```

2. 指定目录(证书过期的话,可以重新执行下面的生成)
```
acme.sh --installcert -d javafroum.cn -d *.javafroum.cn  --keypath    /usr/local/ssl/javafroum.cn.key  --fullchainpath /usr/local/ssl/javafroum.cn.pem
```

## nginx 配置
```
server {
        listen 443;
        server_name javafroum.cn *.javafroum.cn;

        ssl on;
        ssl_certificate      /usr/local/ssl/javafroum.cn.pem;
        ssl_certificate_key  /usr/local/ssl/javafroum.cn.key;

        access_log  logs/javafroum.cn.log  main;
        error_log  logs/javafroum.cn.error.log;

        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Requested-For $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        location / {
                proxy_pass http://127.0.0.1:8080;
                proxy_connect_timeout 600;
                proxy_read_timeout 600;

        }
}

server {
    listen       80;
    server_name blog.javafroum.cn www.javafroum.cn;
    return  301 https://blog.javafroum.cn$request_uri;
}
```


# Springboot单独配置

1. 生成keystore

* 生成 p12 文件(会输入一次密码)
```
            openssl pkcs12 -export -in fullchain.cer -inkey springboot.io.key -out springboot.io.p12
 ```           
2. 根据p12 文件生成 keystore 文件
```
keytool -importkeystore -v  -srckeystore springboot.io.p12 -srcstoretype pkcs12 -srcstorepass 123456 -destkeystore springboot.io.keystore -deststoretype jks -deststorepass 123456
```

* 如果提示警告,可以考虑根据警告的命令,再执行一波
```
keytool -importkeystore -srckeystore springboot.io.keystore -destkeystore springboot.io.keystore -deststoretype pkcs12
```
	
3. springboot配置
```
#ssl
server.ssl.enabled=true
server.ssl.key-store=classpath:ssl/springboot.io.keystore
server.ssl.key-store-type=PKCS12
server.ssl.key-store-password=[key.store的密码]
```