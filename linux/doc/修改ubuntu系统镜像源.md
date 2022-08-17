# 修改ubuntu系统镜像源

使用ubuntu进行系统更新的时候出错，提示不能访问cn.archive.ubuntu.com。  
解决：更换访问源的网址  

1. 编辑源文件

```shell
sudo vim /etc/apt/sources.list
```

2. 将文件中所有的 http://cn.archive.ubuntu.com 更换为 `阿里源` http://mirrors.aliyun.com/ubuntu/  
![图片](../_media/Snipaste_2022-08-17_13-00-06.png ':size=80%')  

3. 执行系统更新

```shell
sudo apt update
```
