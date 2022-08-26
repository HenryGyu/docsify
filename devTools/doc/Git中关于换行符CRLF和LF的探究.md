# Git中关于换行符CRLF和LF的探究

### 背景

`CR`(Carriage Return)代表的是回车，使用`r`来表示；`LF`（Line Feed）代表换行，使用`n`来表示。
由于历史的原因，各操作系统的文本换行符号是不一致的，`windows`系统使用`CRLF`，即`rn`来表示换行，`Uninx`和近些年的`OS X`使用`LF`，即`n`来表示换行，
更早的`Mac`系统则使用`CR`，即`r`来表示换行，不过这系统现在已经很少人用了。

现在问题来了，多人协作开发时，不可能保证每个人使用的系统一致，这就会导致换行编码的不同，虽然说“换行”在人眼看来没有区别，但是`git`面前可就不一样了，
在推拉代码的过程中，如果换行符不一样，我是给你显示文件有修改呢还是直接默认没有任何更改？

好在`git`提供了一个“换行符自动转换”功能帮我们处理了这个问题。默认情况下，`git`在远程仓库保存代码使用`Unix风格`，
即换行符统一使用`LF`模式（`n`），在推-拉代码的过程中，则有以下的规则：

*   提交代码时，`git`会将文本文件中的换行符转换为`LF`模式，这个过程也叫`标准化过程`；
*   拉取代码时，`git`会将试图将仓库中的代码转换为`CRLF`模式，这个过程也叫`转换`；

通过这样的`拉-推`自动转换，`git`不仅保持了远程仓库代码的一致性（Unix风格），而且保证了本地文件的兼容性（Windows系统）。

### Git本地个性化配置

既然有默认配置，那就有个性化的选择，`git`提供了`core.autocrlf`和`core.safecrlf`两个配置项来供开发者自由配置。两者都支持`全局`、`本地`设置。

```shell
git config --global core.autocrlf [true | input | false]   #全局设置
git config --local core.autocrlf [true | input | false]    #针对本项目设置

git config --global core.safecrlf [true | warn | false]    #全局设置
git config --local core.safecrlf [true | warn | false]     #针对本项目设置
```

**core.autocrlf**

`core.autocrlf`有三个值，默认是`true`。这个配置用来决定要不要转换，怎么转换。

```shell
# 提交时转换为LF，检出时转换为CRLF
git config --global core.autocrlf true

# 提交时转换为LF，检出时不转换
git config --global core.autocrlf input

# 提交检出均不转换
git config --global core.autocrlf false
```
> - 第一行：当你将文件添加到暂存区时，Git可以通过设置core.autocrlf将CRLF行结尾自动转换为LF来处理这类问题。
如果您在Windows机器上，将其设置为true，那么当您pull代码时，将LF结尾转换为CRLF。
> - 第二行：如果您使用的是Linux或Mac系统，那么当您pull文件时，您不希望Git自动转换它们;
然而，如果你的Windows猪队友把未经处理带CRLF格式的文件push到远程代码库，那么你可能希望Git来自动解决这个问题。
可以通过将core.autocrlf设置为input来告知Git将CRLF转换为LF。
> - 第三行：如果你是Windows程序员，只执行一个Windows项目或者所团队都是用windows系统甚至服务器都用windows，
那么您可以关闭此功能，通过将配置值设置为false，将回车记录在存储库中。

在Git中，可以通过以下命令来显示当前你的Git中采取哪种方式对待换行符：

```shell
git config core.autocrlf
```

**core.safecrlf**

`core.safecrlf`也有三个值，默认是`false`。这个配置用来决定`add`代码时是否禁止提交混合换行符的文本文件。

```shell
# 禁止提交混合换行符的文本文件(git add 的时候会被拦截，提示异常)
git config --global core.safecrlf true

# 提交混合换行符的文本文件的时候发出警告，但是不会阻止 git add 操作
git config --global core.safecrlf warn

#  默认配置，不禁止提交混合换行符的文本文件
git config --global core.safecrlf false
```

### Git仓库配置

上面的两个配置文件都是个人在本机的个性化设置，如果对于比较大的项目，多人协作的情况下不可能让每个人都去更改一下自己的配置，
这个时候我们就可以在项目的根目录下添加一个`.gitattributes`文件，它的优先级高于`core.autocrlf`配置，它类似于`.gitignore`文件，随提交修改生效，
一个项目中可以维持一份相同的配置。当然，`.gitattributes`文件可不只有配置换行符，还有一些其他的复杂配置。

```git
*           text=auto
# 文件的行尾自动转换。如果是文本文件，则在文件入Git库时，行尾自动转换为LF。如果已经在入Git库中的文件的行尾是GRLF，则文件在入Git库时，不再转换为LF。

*.txt       text
# 对于.txt文件，标记为文本文件，并进行行尾规范化。

*.jpg       -text
# 对于`.jpg`文件，标记为非文本文件

*.vcproj    text eol=crlf
# 对于.vcproj文件，标记为文本文件，在文件入Git库时进行规范化，行尾转换为LF。在检测到出工作目录时，行尾自动转换为GRLF。

*.sh        text eol=lf
# 对于sh文件，标记为文本文件，在文件入Git库时进行规范化，即行尾为LF。在检出到工作目录时，行尾也不会转换为CRLF（即保持LF）。

*.py        eol=lf
# 对于py文件，只针对工作目录中的文件，行尾为LF。
```
