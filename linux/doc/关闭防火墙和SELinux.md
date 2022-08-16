# 关闭防火墙和SELinux

> ### 防火墙（firewalld）

```shell
# 临时关闭防火墙
systemctl stop firewalld

# 防火墙开机不自启动
systemctl disable firewalld

# 临时打开防火墙
systemctl start firewalld

# 防火墙开机自启动
systemctl enable firewalld

# 查看防火墙状态
systemctl status firewalld
```

---

> ### SELinux

<font color='red'> 需安装了selinux-utils，才可以设置。没有该软件包，则无需设置。</font>

```shell
# 临时关闭SELinux
setenforce 0

# 临时打开SELinux
setenforce 1

# SELinux开机不自启动
# 编辑/etc/selinux/config文件，将SELINUX的值设置为disabled

# 查看SELinux状态
getenforce
```
