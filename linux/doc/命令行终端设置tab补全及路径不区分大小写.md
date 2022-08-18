# 命令行终端设置tab补全及路径不区分大小写

### 1. 命令补全

**安装命令补全工具（centos用yum安装，ubuntu用apt安装）**

```shell
apt install bash-completion
```

---

### 2. 设置路径不区分大小写
**编辑文件**

```shell
vim /etc/inputrc
```

**行尾加入如下命令**

```shell
set completion-ignore-case on
```

---

### 3. 重启

```shell
reboot
```
