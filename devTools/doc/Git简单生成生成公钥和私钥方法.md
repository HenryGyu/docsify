# Git简单生成生成公钥和私钥方法

**Git配置**

Git安装完之后，需做最后一步配置。打开终端，分别执行以下两句命令：  
git config --global user.name "用户名"  
git config --global user.email "邮箱"

---

**SSH配置**

1. 打开终端

2. 执行生成公钥和私钥的命令：ssh-keygen -t rsa 并按回车3下。会在用户主目录的`.ssh`文件夹里面生成一个私钥`id_rsa`和一个公钥`id_rsa.pub`。
> 为什么按三下，是因为有提示你是否需要设置密码，如果设置了每次使用Git都会用到密码，一般都是直接不写为空，直接回车就好了

3. 执行查看公钥的命令：cat ~/.ssh/id_rsa.pub

