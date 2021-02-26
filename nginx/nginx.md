# 简介
Nginx 是高性能的 HTTP 和反向代理的服务器，处理高并发能力是十分强大的，特点是占有内存少，并发能力强，能经受高负载的考验,有报告表明能支持高达 50,000 个并发连接数。Nginx 可以作为静态页面的 web 服务器，同时还支持 CGI 协议的动态语言，比如 perl、php等。但是不支持 java。Java 程序只能通过与 tomcat 配合完成。支持热部署，在软件升级的情况下都能使用。

# centos 7安装安装和常用操作

## 安装
1. 添加Nginx到YUM源
添加CentOS 7 Nginx yum资源库,打开终端,使用以下命令
```
sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

2. 安装Nginx
在你的CentOS 7 服务器中使用yum命令从Nginx源服务器中获取来安装Nginx
```
sudo yum install -y nginx
```

3. 启动Nginx
刚安装的Nginx不会自行启动。运行Nginx
```
sudo systemctl start nginx.service
```

4. CentOS 7 开机启动Nginx
```
sudo systemctl enable nginx.service
```

5. Nginx配置信息
+ +网站文件存放默认目录
```
/usr/share/nginx/html
```
+ 网站默认站点配置
```
/etc/nginx/conf.d/default.conf
```
+ 自定义Nginx站点配置文件存放目录
```
/etc/nginx/conf.d/
```
+ Nginx全局配置
```
/etc/nginx/nginx.conf
```
+ Nginx启动
```
nginx -c nginx.conf
```

6. Linux查看公网IP
您可以运行以下命令来显示你的服务器的公共IP地址:
```
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```

7. 卸载NGINX
```
yum remove nginx
which nginx
```

8. 常用命令
+ 开始
```
sudo service nginx start
```
+ 停止
```
sudo service nginx stop
```
+ 重启
```
sudo service nginx restart
```
+ 重新加载配置文件(改了配置，重新加载配置文件会生效)
```
sudo service nginx reload
```
+ 查看nginx 进程状况
```
ps -ef | grep nginx
```

## 配置文件介绍
nginx.conf由3部分组成，全局块，events块，http块
```
#全局块
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

#ecents块
events {
    worker_connections  1024;
}

#http块
http {

    #httphttp全局块
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    #server快  存在多个  用配置文件的形式分开
    #这里引入配置文件  方便管理和配置，默认安装下 里面有个默认配置
    include /etc/nginx/conf.d/*.conf;
}

```
默认server 配置,在conf.d里面有个default.conf,既是server配置
```
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```


### 全局块
从配置文件开始到 events 块之间的内容，主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配
置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以
及配置文件的引入等。
比如上面第一行配置的：
```
worker_processes  1;
```
这是 Nginx 服务器并发处理服务的关键配置，worker_processes 值越大，可以支持的并发处理量也越多，但是
会受到硬件、软件等设备的制约

### event块
```
events {
    worker_connections  1024;
}
```
events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。
上述例子就表示每个 work process 支持的最大连接数为 1024.
这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置。


### http块
这部分是配置中最频繁的一部分，代理，缓存和日志等绝大多数功能和第三方模块的配置都在这里。
**注意：http块包含两部分http全局块和server块**

#### http全局块
http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。

#### server块
这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。  
每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机。
而每个 server 块也分为全局 server 块，以及可以同时包含多个 locaton 块。
1. 全局 server 块  
最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。
2. location 块  
一个 server 块可以配置多个 location 块。
这块的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。

## 防火墙相关设置
1. 查看开放的端口号 
```
firewall-cmd --list-all
```
2. 设置开放的端口号 
```
//firewall-cmd --add-service=http –permanent
sudo firewall-cmd --add-port=80/tcp --permanent
```
3. 重启防火墙
```
firewall-cmd –reload
```


# 反向代理
## 概念介绍
1. 正香代理
需要在客户端配置代理服务器，通过代理服务器去访问网站

```

客服端  ---------不能访问--------------->网站（goole）
    -                                       -
       -                                -
        访问代理服务器            代理服务器访问网站
                -               -
                    代理服务器
````

2. 反向代理
反向代理，其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器 IP 地址。
```

                                          真实服务器1（8080）
                                       -
                                    -
客服端------>反向代理服务器（80） ------>      真实服务器2（8081）
                                    -
                                       - 
                                           真实服务器3（8082）
```

说明：
    暴露的是80端口，真实服务器的端口可ip都可以不暴露,客服端并不到服务端是否用了代理服务器，和真实服务器的 信息。

## 配置例子1
nginx 通过监听80端口，反向代理到服务器8080端口上去
1. 将server的监听端口改为80（默认是80端口）
2. 将location添加proxy_pass参数  
例子
```
server {
    listen       80;
    server_name  location;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        #指向服务器的地址
        proxy_pass http://127.0.0.1:8080;
        index  index.html index.htm;
    }
}
```

## 配置例子2
如果我们要通过路径上的参数区分，指向不同的应用。这可以像下面这种配置
```
server {
    listen       80;
    server_name  location;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location ~/api/ {
        root   /usr/share/nginx/html;
        #指向服务器的地址
        proxy_pass http://127.0.0.1:8080;
        index  index.html index.htm;
    }

     location ~/manager/ {
        root   /usr/share/nginx/html;
        #指向服务器的地址
        proxy_pass http://127.0.0.1:8081;
        index  index.html index.htm;
    }
}
```
这样就实现了：  
访问http://127.0.0.1/api/index.html 就会跳转到8080的运用上
访问http://127.0.0.1/manager/index.html 就会跳转到8081的运用上

## localtion 后面的正则表达式说明
```
locatioon [ = | ~ |~* | ……~] url{

}
```
1. = ：用于不含正则表达式的 uri 前，要求请求字符串与 uri 严格匹配，如果匹配
成功，就停止继续向下搜索并立即处理该请求。
2. ~：用于表示 uri 包含正则表达式，并且区分大小写。
3. ~*：用于表示 uri 包含正则表达式，并且不区分大小写。
4. ^~：用于不含正则表达式的 uri 前，要求 Nginx 服务器找到标识 uri 和请求字
符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location 
块中的正则 uri 和请求字符串做匹配。
** 注意：如果 uri 包含正则表达式，则必须要有 ~ 或者 ~* 标识。**



# 负载均衡
增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的
情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的负
载均衡.

## 负载均衡配置

1. 在http 全局块里面添加upstream
2. 更改server_name 为请求地址
3. 更改proxy_pass的地址，为我们配置的服务列表名

```
#配置服务器列表的地方
upstream serverlist{
    server 192.168.0.110:8080;
    server 192.168.0.110:8081;
}

server {
    listen       80;
    server_name  192.168.1.110;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location ~/api/ {
        root   /usr/share/nginx/html;
        #这里改为上面我们起的名字
        proxy_pass http://serverlist;
        index  index.html index.htm;
    }
}
```

## nginx的分配策略
1. 轮询（默认）  
上面的配置方式即是按照轮询的方式选择服务器的。这也是nginx默认的方式。其特点是：每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。  
2。 weight  
weight 代表权,重默认为 1,权重越高被分配的客户端越多
指定轮询几率，weight 和访问比率成正比，用于后端服务器性能不均的情况。配置方式。
```
upstream serverlist{
    server 192.168.0.110:8080 weight=5;
    server 192.168.0.110:8081 weight=10; 
}
```
3. ip_hash  
每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 的问题。配置方式。
```
upstream serverlist{
    ip_hash;
    server 192.168.0.110:8080;
    server 192.168.0.110:8081;
}
```
4. fair（第三方）
按后端服务器的响应时间来分配请求，响应时间短的优先分配。配置方式。
```
upstream serverlist{
    server 192.168.0.110:8080;
    server 192.168.0.110:8081;
    fair;
}
```
# 动静分离
为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。

说明：
Nginx 动静分离简单来说就是把动态跟静态请求分开，不能理解成只是单纯的把动态页面和
静态页面物理分离。严格意义上说应该是动态请求跟静态请求分开，可以理解成使用 Nginx 
处理静态页面，Tomcat 处理动态页面。动静分离从目前实现角度来讲大致分为两种，
一种是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案；
另外一种方法就是动态跟静态文件混合在一起发布，通过 nginx 来分开。
通过 location 指定不同的后缀名实现不同的请求转发。通过 expires 参数设置，可以使
浏览器缓存过期时间，减少与服务器之前的请求和流量。具体 Expires 定义：是给一个资
源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，
所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件，
不建议使用 Expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个 URL，发送
一个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码
304，如果有修改，则直接从服务器重新下载，返回状态码 200。

## 配置
```
#指向动态服务器
location /www/ {
    root /data/;
    index index.html index.htm;
}

#指向静态资源服务器
location /image/ {
    root /data/;
    # 开启目录浏览   也就是在浏览器上访问目录  有相应的列表
    autoindex on;
}
```

不同的路径，匹配到不同的资源，在做项目的时候，最好把静态资源统一放到一个文件夹下（例如: static）

# 高可用
在上面的负载均衡中，我们可以有多个应用服务器，一个宕机了，可以自动剔除，单nginx宕机了，服务就无法进行了。高可用计算为了解决nginx宕机的情况下，能继续使用服务。解决这个的一般思路计算准备一个备用服务器。
```                             keepalived(需要的软件)
                            192.168.0.220
                           主nginx(master)---------------> 
                     -                                        应用1
                    -                       -       -
客服端-------->虚里ip 192.168.0.110             -   
                    -                       -       -
                      -                                       应用2
                         备nginx(BACKUP)----------------->
                         192.168.0.120 
                         keepalived（需要的软件）


```
需要的环境
* 两台nginx 服务器
* 两台nginx 服务器上需要安装keepalived
* 需要虚拟ip

1. 安装keepalived
安装keepalived
```
yum install  keepalived -y
```
查看安装是否成功
```
rpm -q -a keepalived
```

这里都不详细解释怎么配置了，需要的时候在goole吧 



# nginx的原理
1. 1个manster线程，可以设置多个worker线程。worker采用抢的 形式获取任务，和redis一样采用多路复用机制。
2. worker数量一般设置为cpu的核心数比较好
3. 发一个请求有几个连接数，2个或者4个，如果每家tomcat，就是2个，有就是4个
4. 最大连接数计算
nginx 有一个 master，有四个 woker，每个 woker 支持最大的连接数 1024，支持的
最大并发数是多少？
* 普通的静态访问最大并发数是： worker_connections * worker_processes /2，
* 如果是 HTTP 作 为反向代理来说，最大并发数量应该是 worker_connections *worker_processes/4



# https配置
https://www.oschina.net/news/94188/acme-v2-and-wildcard-certificate-support-is-live
https://my.oschina.net/kimver/blog/1634575
https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E

## Nginx反向代理http的tomcat
	1,执行
		curl https://get.acme.sh | sh


	2,执行
		source ~/.bashrc
	
	3,执行
		# 阿里云后台的密钥
		export Ali_Key="1858118"
		export Ali_Secret="1858118"
	
		# 填写自己的域名
		acme.sh --issue --dns dns_ali -d javafroum.cn -d *.javafroum.cn
		
		* acme.sh比certbot的方式更加自动化,省去了手动去域名后台改DNS记录的步骤,而且不用依赖Python
		* 第一次成功之后,acme.sh会记录下App_Key跟App_Secret,并且生成一个定时任务,每天凌晨0：00自动检测过期域名并且自动续期
		* 对这种方式有顾虑的,请慎重,不过也可以自行删掉用户级的定时任务,并且清理掉~/.acme.sh文件夹就行
	
	4,在证书生成目录执行
	```
	    acme.sh --installcert -d javafroum.cn -d *.javafroum.cn  \
	    --keypath       /usr/local/ssl/javafroum.cn.key  \
	    --fullchainpath /usr/local/ssl/javafroum.cn.pem
	````
		
		* 会把key和pem生成到指定的目录
	
	5,配置nginx
```
server {
        listen 443;
        server_name javafroum.cn www.javafroum.cn;

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
    server_name  javafroum.cn www.javafroum.cn;
    return  301 https://javafroum.cn$request_uri;
}
```
	6,acme.sh 源码
		#!/usr/bin/env sh
	
		#https://github.com/Neilpang/get.acme.sh
	
		_exists() {
		  cmd="$1"
		  if [ -z "$cmd" ] ; then
			echo "Usage: _exists cmd"
			return 1
		  fi
		  if type command >/dev/null 2>&1 ; then
			command -v $cmd >/dev/null 2>&1
		  else
			type $cmd >/dev/null 2>&1
		  fi
		  ret="$?"
		  return $ret
		}
	
		if _exists curl && [ "${ACME_USE_WGET:-0}" = "0" ]; then
		  curl https://raw.githubusercontent.com/Neilpang/acme.sh/master/acme.sh | INSTALLONLINE=1  sh
		elif _exists wget ; then
		  wget -O -  https://raw.githubusercontent.com/Neilpang/acme.sh/master/acme.sh | INSTALLONLINE=1  sh
		else
		  echo "Sorry, you must have curl or wget installed first."
		  echo "Please install either of them and try again."
		fi

## Springboot单独配置
	1,(在证书生成目录)生成keystore
		* 生成 p12 文件(会输入一次密码)
			openssl pkcs12 -export -in fullchain.cer -inkey springboot.io.key -out springboot.io.p12
			
		* 根据p12 文件生成 keystore 文件
			keytool -importkeystore -v  -srckeystore springboot.io.p12 -srcstoretype pkcs12 -srcstorepass 123456 -destkeystore springboot.io.keystore -deststoretype jks -deststorepass 123456
		
			* 如果提示警告,可以考虑根据警告的命令,再执行一波
				keytool -importkeystore -srckeystore springboot.io.keystore -destkeystore springboot.io.keystore -deststoretype pkcs12


	2,springboot配置
		#ssl
		server.ssl.enabled=true
		server.ssl.key-store=classpath:ssl/springboot.io.keystore
		server.ssl.key-store-type=PKCS12
		server.ssl.key-store-password=[key.store的密码]


------------------------------------
手动安装单域名证书					|
------------------------------------

1,clone 
	git clone git@github.com:certbot/certbot.git

2,生成证书
	* 在当前服务器生成(standalone)建议
		./letsencrypt-auto certonly --standalone --email [邮箱] -d [域名] -d [域名]

	 * 非当前服务器上生成(manual)
		./letsencrypt-auto certonly --manual --email [邮箱] -d [域名] -d [域名]
	
		* 需要在服务器的上安装可访问的,脚本提供的文本
			
			CjhdUm0L4oQU0ZHg7F7832FtFweUPlRFJs0LxJGx_qg.-ielpqOUtyZI_Q0f9xYi8-Bj57TsuD5y4mGIMxW9GwM			文本
			http://[域名]/.well-known/acme-challenge/CjhdUm0L4oQU0ZHg7F7832FtFweUPlRFJs0LxJGx_qg			访问地址
	
	* 执行期间会与控制台有两次交互,第一次输入A,表示同意协议,第二次输入YES,表示允许他们收集你的邮箱
	* 创建期间,必须保证80端口不被占用,脚本会在80端口启动http服务来验证域名所属(standalone)
	* 创建成功后会在  /etc/letsencrypt/ 下生成证书文件
	* 如果遇到因为 pip 带来的异常,尝试删除(建议重命名处理) ~/.pip/pip.conf 文件
	* 记得安全组/防火墙开放80端口

3,证书续约(需要在生成证书的服务器上进行)
	./letsencrypt-auto renew
		

		Saving debug log to /var/log/letsencrypt/letsencrypt.log
	
		- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
		Processing /etc/letsencrypt/renewal/springcloud.io.conf(处理/etc/letsencrypt/renewal/springcloud.io.conf)							
		- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
		Cert not yet due for renewal(证书还没有到期)														
	
		- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
	
		The following certs are not due for renewal yet:(下列证书尚未到期续期)									
		  /etc/letsencrypt/live/springcloud.io/fullchain.pem expires on 2018-11-29 (skipped)
		No renewals were attempted.(没有尝试更新)
		- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

# nginx配置
		ssl on;
		ssl_certificate      /etc/letsencrypt/live/[域名]/fullchain.pem
		ssl_certificate_key  /etc/letsencrypt/live/[域名]/privkey.pem

# 转换为 tomcat 的格式
	1,进入目录  /etc/letsencrypt/live/[域名]/ 执行
		openssl pkcs12 -export -in fullchain.pem -inkey privkey.pem -out [域名].p12
	
		* 生成 p12 文件(会输入一次密码)
	
	2,根据p12 文件生成 keystore 文件
		keytool -importkeystore -v  -srckeystore [域名].p12 -srcstoretype pkcs12 -srcstorepass [p12文件的密码] -destkeystore [域名].keystore -deststoretype jks -deststorepass [keytroe的密码]
	
		* 如果提示警告,可以考虑复制警告的命令,再执行一波
				keytool -importkeystore -srckeystore [域名].keystore -destkeystore [域名].keystore -deststoretype pkcs12

------------------------------------
手动安装泛域名证书					|
------------------------------------
1,下载
	wget https://dl.eff.org/certbot-auto
	
	chmod 775 certbot-auto
	
	* 也可采用certbot官方 yum安装方式

2,执行
	./certbot-auto certonly  -d *.javaweb.io -d javaweb.io --manual --email 747692844@qq.com --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory

	* 其实就是跟上面的那种方法一样,采用的是,非当前服务器上生成(manual)

3,添加TXT解析记录到dns服务器,添加完成后,等待证书生成,生成后	
	* 创建成功后会在  /etc/letsencrypt/ 下生成证书文件
	* 跟手动创建单域名证书一样的
	
4,更新证书
	./certbot-auto renew

	* ??失败了,可以尝试重新执行生成操作


------------------------------------
在线证书转换工具					|
------------------------------------

	https://myssl.com/