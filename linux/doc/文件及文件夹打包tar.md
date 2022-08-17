# 文件及文件夹打包tar

概述
---

`tar` 命令可用于将多个文件和目录一起打包成一个文件，但不压缩。如果要压缩，可以使用 `gzip`、`bzip2` 这样的压缩工具。

![图片](../_media/Snipaste_2022-08-17_11-47-56.png ':size=80%')

语法
---

该命令的语法如下：

```
tar [选项] [文件]
```

该命令支持的选项有：

| 选项 | 说明 |
| :-- | :-- |
| \-c | 创建压缩文件 |
| \-x | 解开压缩文件 |
| \-t, --list | 列出压缩包中的文件列表 |
| \-z | 用 gzip 格式压缩或解压 |
| \-j | 用 bzip2 格式压缩或解压 |
| \-v | 显示命令的执行过程 |
| \-f | 目标文件名 |
| \-p | 保留原始的权限与属性 |
| \-P | 使用绝对路径来压缩 |
| \-C | 解压包中所有文件到指定目录 |
| –delete | 从包中删除某个文件 |
| \-r, --append | 将文件追加到包中 |
| \-u, --update | 更新包中的文件 |

> 注：该命令常用的选项就是打包 `-cvf` 和解包 `-xvf`，以及同其他压缩工具进行的压缩与解压缩（如使用 `gzip` 工具压缩是 `-zcvf` 与 `zxvf`）。

使用
---

#### 打包指定文件和目录【★★★★★】

如果想要将指定文件和目录进行打包，命令格式如下：

```
# 语法
tar -cvf tar包名 待压缩的文件或目录
# 示例
tar -cvf test.tar
```

![图片](../_media/Snipaste_2022-08-17_11-50-38.png ':size=80%')

> 注：`-c` 表示创建压缩文件；`-v` 表示显示压缩过程；`-f` 放在最后一位，表示要压缩的包的名称。

#### 列出 `tar` 包中文件列表【★★★★★】

如果要查看压缩包中的文件列表，则可以使用 `-t` 选项：

```
# 语法
tar -tvf tar包名
# 示例
tar -tvf test.tar
```

![在这里插入图片描述](../_media/Snipaste_2022-08-17_11-56-05.png ':size=80%')

#### 追加文件到 `tar` 包中

如果要对已经创建的 `tar` 包追加文件，则可以通过 `-r` 选项：

```
# 语法
tar -rvf tar包名 待追加文件
# 示例
tar -rvf test.tar 123.log
```

![图片](../_media/Snipaste_2022-08-17_11-54-11.png ':size=80%')

#### 解压包中文件到当前目录【★★★★★】

如果要解压包中原有的文件和目录到当前目录，则需要通过 `-x` 选项：

```
# 语法
tar -xvf tar包名
# 示例
tar -xvf test.tar
```

![图片](../_media/Snipaste_2022-08-17_11-57-23.png ':size=60%')

#### 解压包中文件到指定目录【★★★★★】

如果要解压包中所有文件到指定目录，则需要使用 `-C` 选项：

```
# 语法
tar -xvf tar包名 -C 指定解压目录
# 示例
tar -xvf test.tar -C
```

![图片](../_media/Snipaste_2022-08-17_11-59-54.png ':size=80%')

#### 使用 `gzip` 压缩打包【★★★★★】

`tar` 只具有打包的效果，但并不具有压缩的能力，而如果要压缩可以使用 `gzip` 工具对其压缩，即使用 `-z` 选项。这种 `tar` 文件的扩展名是 `.tar.gz` 或 `.tgz`。

```
# 语法
tar -zcvf 压缩包名 待压缩文件或目录
# 示例
tar -zcvf test.tar.gz 123.log inputrc.sh log.txt test/
```

![图片](../_media/Snipaste_2022-08-17_12-02-24.png ':size=80%')

> 注：如果要解压缩则使用 `tar -zxvf 压缩包名` 命令。

![图片](../_media/Snipaste_2022-08-17_12-04-21.png ':size=60%')

#### 使用 `bzip2` 压缩打包

如果使用 `bzip2` 工具对其压缩，即使用 `-j` 选项。这种 `tar` 文件的扩展名是 `.tar.bz2` 或 `.tbz`。

```
# 语法
tar -jcvf 压缩包名 待压缩文件或目录
# 示例
tar -jcvf test.tar.bz2 123.log inputrc.sh log.txt test/
```

![图片](../_media/Snipaste_2022-08-17_12-07-31.png ':size=80%')

> 注：如果要解压缩则使用 `tar -jxvf 压缩包名` 命令。

![图片](../_media/Snipaste_2022-08-17_12-08-54.png  ':size=60%')

#### 使用绝对路径

如果我们仍然用上面的选项对绝对路径进行打包。就会报错：`tar: Removing leading /' from member names`。

![图片](../_media/Snipaste_2022-08-17_12-10-18.png ':size=80%')  
但其实压缩包是生成了的，但为了避免出现这个问题，我们可以加上 `-P` 选项，就能使用绝对路径来压缩打包了。

```
# 语法，注意 P 要在 f 的前面，因为 -f 选项跟文件名
tar -zcvPf 压缩包名 待压缩文件的绝对路径
# 示例
tar -zcvPf /home/zhangsan/hello.txt.tar.gz /home/zhangsan/hello.txt
```

![图片](../_media/Snipaste_2022-08-17_12-11-33.png ':size=80%')

---

本文转自 [https://blog.csdn.net/cnds123321/article/details/124932309](https://blog.csdn.net/cnds123321/article/details/124932309)，如有侵权，请联系删除。
