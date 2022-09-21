# Linux安装JDK

1、使用`java -version`测试系统是否已安装JDK。若已安装则先卸载或不必再安装。

```shell
# 已centos为例，如果操作系统不是最小安装，会默认安装openjdk
rpm -qa | grep java
# 删除系统预装jdk
rpm -e --nodeps `rpm -qa | grep java`
```

2、上传`jdk-8u301-linux-x64.tar.gz`并解压缩到`/home/software/jdk1.8.0_301`

```shell
# 先上传到 /home/software 目录下，再执行解压命令
tar -zxvf /home/software/jdk-8u301-linux-x64.tar.gz -C /home/software
```

3、编辑环境变量配置（给到所有用户）

```shell
vim /etc/profile
```

4、profile文件末尾追加

```shell
export JAVA_HOME=/home/software/jdk1.8.0_301
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

5、刷新配置

```shell
source /etc/profile
```

6、使用`java -version`检查jdk是否生效
