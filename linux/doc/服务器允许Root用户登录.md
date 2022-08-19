# 服务器允许Root用户登录

**1. 重置root密码**

```shell
sudo passwd root
```

**2. 修改ssh配置文件**

sudo vim /etc/ssh/sshd_config后，进入配置文件中修改`PermitRootLogin`的默认值为yes

![](../_media/Snipaste_2022-08-18_14-03-15.png ':size=40%')

**3. 重启ssh守护进程**

```shell
sudo service ssh restart
```
