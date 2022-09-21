# Mac安装JDK

1、先下载安装包执行安装。安装后，JDK主目录为`/Library/Java/JavaVirtualMachines/jdk1.8.0_301.jdk/Contents/Home`

2、编辑环境变量配置

```shell
vim ~/.bash_profile
```

3、文件末尾追加

```shell
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_301.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH
```

4、刷新配置

```shell
source ~/.bash_profile
```

5、使用`java -version`检查jdk是否生效
