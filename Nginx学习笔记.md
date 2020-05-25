##### Nginx基础回顾

###### 是什么

Nginx是一个高性能的HTTP和反向代理web服务器，核心特点是占有内存少，并发能力强

###### 正向代理

在浏览器中配置代理服务器的相关信息，通过代理服务器访问目标网站

###### 反向代理

浏览器将请求发送给反向代理服务器（如Nginx)，由反向代理服务器选择目标服务器提供服务并获取结果，最终再返回给客户端浏览器

###### 负载均衡服务器

根据负载均衡算法选择目标服务器来响应客户端请求。为了解决高负载的问题

###### 动静分离

静态资源交与nginx处理，动态资源由web服务器处理

###### 特点

- 跨平台
- 上手容易，配置简单
- 高并发，性能好
- 稳定

###### 安装

- 上传nginx安装包到linux服务器，下载地址http://nginx.org 

- 安装nginx依赖，pcre\openssl\gcc\zlib

  yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel 

- 解压 tar -xvf nginx-1.17.8.tar

- cd nginx-1.17.8

- ./configure

- make

- make install，完毕后在/usr/local/下会产生一个nginx目录

- 进入sbin目录，启动 

  cd nginx/sbin

  ./nginx

###### 主要命令

./nginx 启动

./nginx -s stop 终止

./nginx -s reload 重新加载nginx.conf配置文件

##### Nginx核心配置文件解读

包含三块内容：全局块、events块、http块

全局块：从配置文件开始到events块之间的内容，此处的配置影响nginx服务器整体的运行，比如worker进行的数量，错误日志的位置等

events：主要影响nginx服务器与用户的网络连接

http:虚拟主机的配置、监听端口的配置、请求转发、反向代理、负载均衡等

```conf
#运行用户
#user  nobody;
#worker进程数量，通常设置为和cpu数量相等
worker_processes  1;

#全局错误日志及pid文件位置
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
	#单个worker进行的最大并发连接数
    worker_connections  1024;
}


http {
	#引入mime类型定义文件
    include       mime.types;
    default_type  application/octet-stream;
	#设定日志格式
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;
	#连接超时时间
    #keepalive_timeout  0;
    keepalive_timeout  65;
	#开启gzip压缩
    #gzip  on;

    server {
    	#监听的端口
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
		#默认请求
        location / {
            root   html; #默认的网站根目录位置
            index  index.html index.htm;#首页
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        # 错误提示页面
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
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


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```



##### 反向代理

![image-20200525215631280](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200525215631280.png)

##### 负载均衡

- 轮询(默认策略)

  ```
  upstream lagouServer{  
  server 111.229.248.243:8080;  
  server 111.229.248.243:8082; 
  }
  location /abc {  
  proxy_pass http://lagouServer/; 
  }
  
  ```

- weight权值

  ```
  upstream lagouServer{  
  server 111.229.248.243:8080 weight=1;  
  server 111.229.248.243:8082 weight=2; 
  }
  
  ```

- ip_hash

  每个请求按照ip的hash值进行分配，同一个客户端的请求会固定到同一台服务器，可以解决session问题

  ```
  upstream lagouServer{  
  ip_hash;  
  server 111.229.248.243:8080;  
  server 111.229.248.243:8082; 
  }
  ```

##### 动静分离

将动态资源和静态资源交由不同服务器处理，比如nginx+tomcat组合。配置静态资源处理

```
location /static/ {  
	root staticData; 
}
```

##### Nginx底层进程机制刨析

Nginx启动后，以daemon多进程方式在后台运行，包括一个master进程和多个worker进程

- master进程

  主要是管理worker进程

  ​	-接收外界信号向各worker进程发送信号(./nginx -s reload)

  ​	-监控worker进程的运行状态，当worker进程异常退出后master会重新启动新worker

- worker进程

  处理网络请求，同等竞争客户端请求，各进程相互之间是独立的。一个请求只可能在一个worker中处理

  nginx使用互斥锁来保证只有一个worker能处理请求

nginx多进程模型好处

每个worker进程都是独立的，不需要加锁，节省开销，提供了高可用保证

多进程模型为reload热部署机制提供了支撑

