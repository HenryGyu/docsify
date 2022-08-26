# 解决由于CRLF导致shell脚本无法运行

1. [问题来源](#问题来源)
2. [解决方法](#解决方法)

在Windows下编辑的shell脚本在写入时默认的换行符为CRLF，而在Linux中换行符为LF，这可能导致在Windows下编辑的脚本在Linux下运行报错。

### [](#问题来源 "问题来源")问题来源

在Git拉取的项目在Win下编辑并且保存了，结果导致换行符出现了CRLF；
使用Goland远程开发时，将Win中的项目同步push到了云服务器，结果启动脚本报错。

> **什么是CRLF和LF**
>
> CRLF 是 carriage return line feed 的缩写；中文意思是 回车换行  
> LF 是 line feed 的缩写，中文意思是换行
>
> * * *
>
> **换行符\\n和回车符\\r**
>
> 回车符就是回到一行的开头，用符号r表示，十进制ASCII代码是13，十六进制代码为0x0D，回车（return）；  
> 换行符就是另起一行，用n符号表示，ASCII代码是10，十六制为0x0A， 换行（newline）
>
> * * *
>
> **应用情况**
>
> Dos和Windows平台：使用回车（CR）和换行（LF）两个字符来结束一行，回车+换行(CR+LF)，即`\r\n`；  
> Mac 和 Linux平台：只使用换行（LF）一个字符来结束一行，即`\n`；  
> PS: 最早Mac每行结尾是回车CR 即’\\r’，后mac os x 也投奔了 unix
>
> 许多 Windows 上的编辑器会悄悄把行尾的换行（LF）字符转换成回车（CR）和换行（LF），或在用户按下 Enter 键时，
> 插入回车（CR）和换行（LF）两个字符。但是文本编辑器不会。
>
> * * *
>
> **影响**
>
> 一个直接后果是，Unix/Mac系统下的文件在Windows里打开的话，所有文字会变成一行  
> 而Windows里的文件在Unix/Mac下打开的话，在每行的结尾可能会多出一个^M符号  
> Linux保存的文件在windows上用记事本看的话会出现黑点

* * *

### [](#解决方法 "解决方法")解决方法

**① 使用vim编辑器修改文件格式**

在vim编辑栏可以使用`:set ff`查看文件格式：

```shell
:set ff
```

修改文件格式：

```shell
:set ff=unix
```

* * *

**② 使用dos2unix，直接将dos换行符转为unix换行符**

首先可以使用apt或者yum安装dos2unix；  
关于dos2unix命令，参见：[dos2unix命令](https://www.jianshu.com/p/d2e96b2ccab9)  
最简单的用法就是dos2unix直接跟上文件名：`dos2unix file`

* * *

本文转自 [https://jasonkayzk.github.io/2020/05/10/解决由于CRLF导致shell脚本无法运行/](https://jasonkayzk.github.io/2020/05/10/解决由于CRLF导致shell脚本无法运行/)，如有侵权，请联系删除。
