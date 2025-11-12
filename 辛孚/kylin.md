```
/usr/local/sunlogin/bin    vnftp
```

识别U盘  NTFS格式

![image-20250725232456873](C:\Users\24463\AppData\Roaming\Typora\typora-user-images\image-20250725232456873.png)

```
lsblk  
mount -a 存储重新挂载
lsusb 查看是否识别SanDisk的U盘
lshw -C  disk 查看存储硬件信息

#如果发现U盘未挂载 可以创建一个目录用来挂载
mkdir -p /root/usb/{ventoy,vtoyefi}
mount -t exfat /dev/sdb1 /root/usb/ventoy
若报错：需先安装exFAT支持
dnf install exfat-utils fuse-exfat
mount -t exfat /dev/sdb1 /root/usb/ventoy
umount /root/usb/ventoy
若提示“设备忙”，需先关闭占用进程
sudo lsof /root/usb/ventoy  # 查看占用进程
sudo kill -9 [PID]          # 结束进程（替换为实际PID）
sudo kill -9 117988 1935320 1935321
umount /root/usb/ventoy
```

```
./cfw --no-sandbox
```

```
在 Red Hat、CentOS 和 Fedora 上安装 Chrome

打开终端并使用以下命令在基于 Red Hat 的 Linux 发行版（例如 CentOS、Red Hat 和 Fedora）上安装 Google Chrome。

$ wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm
$ sudo dnf localinstall ./google-chrome-stable_current_x86_64.rpm

安装 Chrome 还会将存储库添加到您的包管理器中。使用以下命令使 Chrome 在您的系统上保持最新状态。

$ sudo dnf install google-chrome-stable

如果您决定将来从系统中删除 Chrome，请使用以下命令卸载网络浏览器。

$ sudo dnf remove google-chrome-stable
```

