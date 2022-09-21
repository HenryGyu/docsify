# Mac安装Maven

1、下载Maven包，并解压到`/Users/henry/software/apache-maven-3.8.6`

2、编辑环境变量配置

```shell
vim ~/.bash_profile
```

3、文件末尾追加

```shell
export MAVEN_HOME=/Users/henry/software/apache-maven-3.8.6
export PATH=$MAVEN_HOME/bin:$PATH
```

4、刷新配置

```shell
source ~/.bash_profile
```

5、使用`mvn -version`检查maven是否生效
