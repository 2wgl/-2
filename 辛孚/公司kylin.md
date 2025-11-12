### todesk远程

```shell
todesk:816 744 667   密码:Syspetro123.
向日葵：517670679  密码:admin123 

1. 数据库连接方式
root  kylin  dmdba SYSDBA   用户密码Syspetro123.
```

### VNC安装

```shell
systemctl stop firewalld  #关闭防火墙
yum install -y tigervnc-server #安装
vncserver :1  #启动 1端口 设置密码，提示是否要一个只读的密码，我们选n

windows搜索vnc打开连接 
192.168.0.101:1 --> OK --> 输入设置的密码
```

### 向日葵Linux远程

```shell
https://sunlogin.oray.com/download/linux?type=personal  #向日葵官网下载
https://blog.csdn.net/weixin_53327306/article/details/140854524  #博主
yum install  ./SunloginClient_15.2.0.63064_x86_64.rpm
cp /usr/local/sunlogin/scripts/runsunloginclient.service /etc/systemd/system/

systemctl  daemon-reload
systemctl enable runsunloginclient.service
systemctl start runsunloginclient.service
# 没有图标 图形文件形式打开/usr/local/sunlogin/sunlogin.desktop
```

### 谷歌浏览器

```shell
#在 Red Hat、CentOS 和 Fedora 上安装 Chrome
#打开终端并使用以下命令在基于 Red Hat 的 Linux 发行版（例如 CentOS、Red Hat 和 Fedora）上安装 Google Chrome。
$ wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm
$ sudo dnf localinstall ./google-chrome-stable_current_x86_64.rpm

#安装 Chrome 还会将存储库添加到您的包管理器中。使用以下命令使 Chrome 在您的系统上保持最新状态。
$ sudo dnf install google-chrome-stable
#如果您决定将来从系统中删除 Chrome，请使用以下命令卸载网络浏览器。
$ sudo dnf remove google-chrome-stable
```



```
sed -i 's/User=root/User=todeskuser/' /etc/systemd/system/todeskd.service
useradd -r -s /usr/sbin/nologin todeskuser
```



### dify软件

```shell
#已安装在/root/桌面/dify /usr/local   
#安装docker compose
yum install git
git --version
git clone https://github.com/langgenius/dify.git
cd dify/docker
cp .env.example .env
docker-compose up -d
docker compose ps
http://localhost/install
```

```shell
#ps:运行时出现问题：Error response from daemon: driver failed programming external connectivity on endpoint docker-nginx-1  #该问题是nginx端口冲突导致的，通过vim .env打开.env，在.env中找到：
#EXPOSE_NGINX_PORT是nginx暴露给外部的窗口，更改为其他端口，例如8000、8443，随后再次运行docker compose up -d
```

