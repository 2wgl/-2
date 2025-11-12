```shell
wget https://tengine.taobao.org/download/tengine-3.1.0.tar.gz
yum install gcc openssl-devel pcre-devel zlib-devel -y  #安装依赖
tar -zxvf tengine-3.1.0.tar.gz
cd tengine-3.1.0
./configure --prefix=/usr/local/tengine  --add-module=./modules/ngx_http_upstream_check_module
make && make install

vi /usr/lib/systemd/system/tengine.service
[Unit]n	
Description=Tengine Nginx Web Server
After=network.target
 
[Service]
Type=forking
#PIDFile=/opt/nginx/tengine.pid
ExecStartPre=/usr/local/tengine/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/local/tengine/sbin/nginx
ExecReload=/usr/local/tengine/sbin/nginx -s reload
ExecStop=/usr/local/tengine/sbin/nginx -s quit
PrivateTmp=true
 
[Install]
WantedBy=multi-user.target

systemctl daemon-reload  #重载

vi /usr/local/tengine/conf/nginx.conf  #修改端口
 
    server {
        listen       8081;
        
systemctl start tengine
systemctl status tengine
ps -ef | grep nginx


/usr/local/tengine/sbin/nginx #启动
/usr/local/tengine/sbin/nginx -s quit
```



```shell
/usr/local/tengine/sbin/nginx -s stop
ps aux | grep nginx
rm -rf /usr/local/tengine/
rm -f /usr/lib/systemd/system/tengine.service
systemctl daemon-reload
```

```shell
#8399:8399
docker run -d \
  --name sp-gateway \
  -p 8499:8399 \ 
  -v /root/wyt/_install/nginx/conf/tengine.conf:/usr/local/tengine/conf/nginx.conf:ro \
  -v /root/wyt/_install/nginx/data:/usr/local/tengine/html:ro \
  -v /root/wyt/_install/nginx/logs:/var/log/tengine 
  --ulimit nofile=65536:65536 \
  --privileged=true \
  --cpus=16.0 \
  --memory=64g \
  tengine:3.1.0
```





```shell
FROM        alpine:3.10
COPY  ./tengine-3.1.0.tar.gz  /tmp
RUN   rm -rf /etc/yum.repos.d/*
RUN sed  -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g'   /etc/apk/repositories  && \
    apk add --no-cache gcc libc-dev make  pcre-dev openssl-dev zlib-dev && \
    cd /tmp && \
    tar -zxvf tengine-3.1.0.tar.gz && \
    cd tengine-3.1.0 && \
    ./configure --prefix=/usr/local/tengine  --add-module=./modules/ngx_http_upstream_check_module && \
    make && make install
EXPOSE  80 443
WORKDIR  /usr/local/tengine/html
CMD   ["/usr/local/tengine/sbin/nginx","-g","daemon off;"]
```



```shell
FROM        alpine:3.10
COPY  ./tengine-3.1.0.tar.gz  /tmp
RUN   rm -rf /etc/yum.repos.d/*
COPY  ./CentOS-Base.repo /etc/yum.repos.d
RUN yum install -y  gcc make openssl-devel pcre-devel zlib-devel && \
    yum clean all
RUN cd /tmp && \
    tar -zxvf tengine-3.1.0.tar.gz && \
    cd tengine-3.1.0 && \
    ./configure --prefix=/usr/local/tengine  --add-module=./modules/ngx_http_upstream_check_module && \
    make && make install
EXPOSE  80 443
WORKDIR  /usr/local/tengine/html
CMD   ["/usr/local/tengine/sbin/nginx","-g","daemon off;"]
```



### tengine配置文件

```nginx
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#error_log  "pipe:rollback logs/error_log interval=1d baknum=7 maxsize=2G";

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;
    #access_log  "pipe:rollback logs/access_log interval=1d baknum=7 maxsize=2G"  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
        #access_log  "pipe:rollback logs/host.access_log interval=1d baknum=7 maxsize=2G"  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
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

        # pass the Dubbo rpc to Dubbo provider server listening on 127.0.0.1:20880
        #
        #location /dubbo {
        #    dubbo_pass_all_headers on;
        #    dubbo_pass_set args $args;
        #    dubbo_pass_set uri $uri;
        #    dubbo_pass_set method $request_method;
        #
        #    dubbo_pass org.apache.dubbo.samples.tengine.DemoService 0.0.0 tengineDubbo dubbo_backend;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }

    # upstream for Dubbo rpc to Dubbo provider server listening on 127.0.0.1:20880
    #
    #upstream dubbo_backend {
    #    multi 1;
    #    server 127.0.0.1:20880;
    #}

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









