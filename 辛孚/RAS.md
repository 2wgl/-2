### RAS部署

```shell
#1.部署SP-Ocelot
vi /etc/systemd/system/spocelot.service
#Dockerfile
FROM spbase6
WORKDIR /app
EXPOSE 5000
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8
ENV ASPNETCORE_URLS=http://+:5000
COPY . /app
ENTRYPOINT ["dotnet", "SP.SyspetroOcelot.dll"]

docker build -t spocelot . 
docker run -d -p 13001:13001 --name spocelotapi spocelot 
docker run -d -P --network=host --name spocelotapi spocelot 

#2.部署SP-Ocelotinner
vi /etc/systemd/system/spocelotinner.service
#Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:5.0 
WORKDIR /app 
EXPOSE 14001 
ENV ASPNETCORE_URLS=http://+:14001 
COPY . /app 
ENTRYPOINT ["dotnet", "SP.SyspetroOcelot.dll"] 

docker build -t spocelotinner . 
docker run -d -p 14001:14001 --name spocelotinnerapi spocelotinner 
docker run -d -P --network=host --name spocelotinnerapi spocelotinner 

#3.部署SP-Auth
vi /etc/systemd/system/spauth.service
#Dockerfile
FROM spbase6
WORKDIR /app
EXPOSE 6001
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8
ENV ASPNETCORE_URLS=http://+:6001
COPY . /app
ENTRYPOINT ["dotnet", "SP.Auth.dll"]

docker build -t spauth .
docker run -d -p 6001:6001 --name spauthapi spauth
docker run -d -P --network=host --name spauthapi spauth

#4.部署SP-Sys
vi /etc/systemd/system/spsys.service
#Dockerfile
FROM spbase6
WORKDIR /app
EXPOSE 8310
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8
ENV ASPNETCORE_URLS=http://+:8310
COPY . /app
ENTRYPOINT ["dotnet", "SP.Sys.Web.Entry.dll"]

docker build -t spsys . 
docker run -d -p 8310:8310 --name spsysapi spsys
docker run -d -P --network=host --name spsysapi spsys

#5.部署SP-RAS
vi /etc/systemd/system/spras.service
#Dockerfile
FROM spci62024
WORKDIR /app
# 安装 nano
RUN apt-get update && \
    apt-get install -y nano && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

EXPOSE 8317

ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8
ENV ASPNETCORE_URLS=http://+:8317

COPY . /app

ENTRYPOINT ["dotnet", "SP.MQS.Web.Entry.dll"]

docker build -t spras . 
docker run -d -p 8318:8318 --name sprasapi -v /home/syspetro/site/SPRAS/UploadFiles:/app/UploadFiles spras 
docker run -d -P --network=host --name sprasapi -v /home/syspetro/site/SPRAS/UploadFiles:/app/UploadFiles spras

#6.部署SP-RAS.CI
vi /etc/systemd/system/sprasci.service
FROM spci62024
WORKDIR /app
# 安装 nano
RUN apt-get update && \
    apt-get install -y nano && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

EXPOSE 8317

ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8
ENV ASPNETCORE_URLS=http://+:8317

COPY . /app

ENTRYPOINT ["dotnet", "SP.MQS.CI.Web.Entry.dll"]

docker build -t sprasci . 
docker run -d -p 8319:8319 --name sprasciapi -v /home/syspetro/site/SPRAS/UploadFiles:/app/UploadFiles sprasci 
docker run -d -P --network=host --name sprasciapi -v /home/syspetro/site/SPRAS/UploadFiles:/app/UploadFiles sprasci

#7.部署SP-RAS.Web
vi /etc/nginx/conf.d/webroot.conf #修改nginx配置文件
```

```
static/js/app.*.json   baseURL 修改成主机ip:13001
```



### tengine-ras配置

```nginx

#user nginx;
worker_processes auto;
error_log /usr/local/tengine/logs/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /usr/local/tengine/logs/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    client_header_timeout 1200;
    client_body_timeout   1200;
    send_timeout          1200;
    keepalive_timeout   1200;
    types_hash_max_size 4096;
    client_max_body_size 1000M;

    include             /usr/local/tengine/conf/mime.types;
    default_type        application/octet-stream;

    include /usr/local/tengine/conf.d/*.conf;
     server { 
 	     listen       8082; ## nginx监听端口，对应docker-compose端口映射中右侧的端口 
	     server_name  localhost; 
	     gzip on; 
	     gzip_min_length 1k; 
	     gzip_http_version 1.0; 
	     gzip_comp_level 1; 
	 #   gzip_types text/html text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml; 
 	     gzip_disable "MSIE [1-6]."; 
	     gzip_vary on; 
	     #charset koi8-r; 
  	     #access_log  logs/host.access.log  main; 
   	     location / { 
      		  root  /usr/local/tengine/html; 
      		  index  index.html index.htm; 
  	     }

	    location /test {
                  root  /usr/share/;
                  index  index.html index.htm;
             }
}

}


```



### docker-compose 配置

```yaml
# docker-compose.yml

services:
#    sp-gateway:
#        image: nginx
#        container_name: sp-gateway
#        ports:
#            - "8083:8083"
#        volumes:
#            - /root/Syspetro/site/SPRAS/nginx/conf/nginx.conf:/etc/nginx/nginx.conf:ro
#            - /root/Syspetro/site/SPRAS/nginx:/usr/share/nginx/html:ro
#            - /root/Syspetro/site/SPRAS/nginx/logs:/var/log/nginx
#        logging:
#            options:
#                max-size: "20m"
#                max-file: "5"
#        ulimits:
#            nofile:
#                soft: 65536
#                hard: 65536
#        privileged: true
#        cpus: 16.0
#        mem_limit: "64G"

    spras-tengine-gateway:
      image: tengine:3.1.0
      container_name: spras-tengine-gateway
      ports:
          - "8082:8082"
      volumes:
          - /root/Syspetro/site/SPRAS/tengine/conf/tengine.conf:/usr/local/tengine/conf/nginx.conf:ro
          - /root/Syspetro/site/SPRAS/nginx/data:/usr/local/tengine/html:ro
          - /root/Syspetro/site/SPRAS/tengine/logs:/usr/local/tengine/logs
      logging:
          options:
              max-size: "20m"
              max-file: "5"
      ulimits:
          nofile:
              soft: 65536
              hard: 65536
      privileged: true
      cpus: 16.0
      mem_limit: "64G"

#       sp-ocelot:
#           image: spocelot
#           container_name: sp-ocelot
#           build: /root/wyt/_install/publish/ocelot
#           ports:
#               - "13001:5000"
#           volumes:
#               - /root/Syspetro/site/SPRAS/ocelot/13001/appsettings.json:/app/appsettings.json:ro
#               - /root/Syspetro/site/SPRAS/ocelot/13001/ocelot.json:/app/ocelot.json:ro
#               - /root/Syspetro/site/SPRAS/ocelot/13001/logs:/app/logs
#           logging:
#               options:
#                   max-size: "20m"
#                   max-file: "5"
#           ulimits:
#               nofile:
#                   soft: 65536
#                   hard: 65536
#           privileged: true
#           cpus: 16.0
#           mem_limit: "64G"

#       sp-ocelot-inner:
#           image: spocelot
#           container_name: sp-ocelot-inner
#           build: /root/wyt/_install/publish/ocelot
#           ports:
#               - "14001:5000"
#           volumes:
#               - /root/Syspetro/site/SPRAS/ocelot/14001/appsettings.json:/app/appsettings.json:ro
#               - /root/Syspetro/site/SPRAS/ocelot/14001/ocelot.json:/app/ocelot.json:ro
#               - /root/Syspetro/site/SPRAS/ocelot/14001/logs:/app/logs
#           logging:
#               options:
#                   max-size: "20m"
#                   max-file: "5"
#           ulimits:
#               nofile:
#                   soft: 65536
#                   hard: 65536
#           privileged: true
#           cpus: 16.0
#           mem_limit: "64G"

#       sp-sys:
#           image: spsys
#           container_name: sp-sys
#           build: /root/wyt/_install/publish/sys
#           ports:
#               - "8310:8310"
#           volumes:
#               - /root/wyt/_install/login/sys/appsettings.json:/app/appsettings.json:ro
#               - /root/wyt/_install/login/sys/applicationsettings.json:/app/applicationsettings.json:ro
#               - /root/wyt/_install/login/sys/logs:/app/logs
#           logging:
#               options:
#                   max-size: "20m"
#                   max-file: "5"
#           ulimits:
#               nofile:
#                   soft: 65536
#                   hard: 65536
#           privileged: true
#           cpus: 16.0
#           mem_limit: "64G"

#       sp-auth:
#           image: spauth
#           container_name: sp-auth
#           build: /root/wyt/_install/publish/auth
#           ports:
#               - "6001:6001"
#           volumes:
#               - /root/wyt/_install/login/auth/appsettings.json:/app/appsettings.json:ro
#               - /root/wyt/_install/login/auth/logs:/app/logs
#           logging:
#               options:
#                   max-size: "20m"
#                   max-file: "5"
#           ulimits:
#               nofile:
#                   soft: 65536
#                   hard: 65536
#           privileged: true
#           cpus: 16.0
#           mem_limit: "64G"

    sp-ras:
        image: spras
        container_name: sp-ras
        build: /root/Syspetro/site/SPRAS/public/spras
        ports:
            - "8318:8318"
        volumes:
            - /root/Syspetro/site/SPRAS/spras/appsettings.json:/app/appsettings.json:ro
            - /root/Syspetro/site/SPRAS/spras/applicationsettings.json:/app/applicationsettings.json:ro
            - /root/Syspetro/site/SPRAS/UploadFiles:/app/UploadFiles
            - /root/Syspetro/site/SPRAS/spras/logs:/app/logs
        logging:
            options:
                max-size: "20m"
                max-file: "5"
        ulimits:
            nofile:
                soft: 65536
                hard: 65536
        privileged: true
        cpus: 16.0
        mem_limit: "64G"

    sp-rasci:
        image: sprasci
        container_name: sp-rasci
        build: /root/Syspetro/site/SPRAS/public/sprasci
        ports:
            - "8319:8319"
        volumes:
            - /root/Syspetro/site/SPRAS/sprasci/appsettings.json:/app/appsettings.json:ro
            - /root/Syspetro/site/SPRAS/sprasci/applicationsettings.json:/app/applicationsettings.json:ro
            - /root/Syspetro/site/SPRAS/UploadFiles:/app/UploadFiles
            - /root/Syspetro/site/SPRAS/sprasci/logs:/app/logs
        logging:
            options:
                max-size: "20m"
                max-file: "5"
        ulimits:
            nofile:
                soft: 65536
                hard: 65536
        privileged: true
        cpus: 16.0
        mem_limit: "64G"


```





