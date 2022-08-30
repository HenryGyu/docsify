# Linux下Git的升级或安装

（1）执行命令：

```shell
sudo yum install curl-devel expat-devel gettext-devel \
openssl-devel zlib-devel
```

（2）卸载旧版本git：

```shell
yum remove git
```

（3）下载git：

```shell
wget https://github.com/git/git/archive/v2.24.0.tar.gz
```

（4）解压：

```shell
tar -zxf git-2.24.0.tar.gz
```

（5）进入解压后的git安装目录：

```shell
cd git-2.24.0
```

（6）编译：

```shell
make prefix=/usr/local/git all
```

（7）安装：

```shell
make prefix=/usr/local/git install
```

（8）编辑环境变量配置（给到所有用户）：

```shell
vim /etc/profile
```

（9）profile文件末尾追加：

```shell
export PATH=$PATH:/usr/local/git/bin
```

（10）刷新配置：

```shell
source /etc/profile
```

（11）查看版本：

```shell
git —version
```

---

本文转自 [https://www.cnblogs.com/guopengju/p/11906423.html](https://www.cnblogs.com/guopengju/p/11906423.html)，如有侵权，请联系删除。
