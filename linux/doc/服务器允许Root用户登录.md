# 服务器允许Root用户登录

**1. 重置root密码**

```shell
sudo passwd root
```

**2. 修改ssh配置文件**

`sudo vim /etc/ssh/sshd_config`，进入配置文件中修改`PermitRootLogin`的默认值为yes

![](../_media/Snipaste_2022-08-18_14-03-15.png ':size=40%')

**3. 重启ssh守护进程**

```shell
sudo service ssh restart
```

### 补充

**ubuntu开启22端口**

```shell
# 安装 openssh
sudo apt-get install openssh-server openssh-client
# 启动ssh服务
service ssh start
# 测试
lsof -i:22
```

**解决ubuntu图形化界面无法使用root登录**

`sudo vim /etc/pam.d/gdm-autologin`\
把文件中的 auth required pam_succeed_if.so user != root quiet_success 注释掉

`sudo vim /etc/pam.d/gdm-password`\
把文件中的 auth required pam_succeed_if.so user != root quiet_success 注释掉

最后`passwd root`重置root登录密码
