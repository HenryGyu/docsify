# Linux安装Maven

1、上传`apache-maven-3.8.6-bin.tar.gz`并解压缩到`/home/software/apache-maven-3.8.6`

```shell
# 先上传到 /home/software 目录下，再执行解压命令
tar -zxvf /home/software/apache-maven-3.8.6-bin.tar.gz -C /home/software
```

2、编辑环境变量配置（给到所有用户）

```shell
vim /etc/profile
```

3、profile文件末尾追加

```shell
export MAVEN_HOME=/home/software/apache-maven-3.8.6
export PATH=$MAVEN_HOME/bin:$PATH
```

4、刷新配置

```shell
source /etc/profile
```

5、使用`mvn -version`检查maven是否生效
