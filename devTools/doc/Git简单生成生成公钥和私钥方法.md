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

---

**Git存储库遇到不安全问题**

1. 使用vscode、idea等打开git存储库，遇到下面问题：
> fatal: unsafe repository(xxx is owned by someone else.)  
  To add an exception for this directory, call  
  git config –global –add safe.directory

2. 原因分析：
> git近期进行了版本升级，添加了新的目录安全限制。造成在进行git常规操作时，或在各类编辑器如VSCode中无法发现.git文件

3. 处理方案1：忽略单个目录
```shell
# 哪个目录里有.git文件夹，就通过命令行添加哪个目录，多个目录，多次添加
git config --global --add safe.directory C:/code/your-project
git config --global --add safe.directory C:/code/other-project
```

4. 处理方案2：忽略全部文件夹
```shell
# 可以通过加通配符为*，忽略所有文件夹。
# 需要注意，该处理方法一般适用于只有本人一个用户使用的电脑，确保无其它用户，否则存在安全问题。
git config --global --add safe.directory "*"
```

5. 相关升级

本次安全升级的名称为CVE-2022-24765，主要想防范在多用户主机上，通过创建上级目录的方式，进行git配置的篡改。

原有的git机制是，如果本级目录下没有.git目录，它会向上级目录（父级）查找.git目录，直到查找到为止。这种机制下，如果有恶意人员借助共享目录的权限，在最上级目录创建.git文件，可能导致用户误操作在非项目目录中操作git时，将会使用恶意人员部署的git配置。

所以这次git增加了限制，在逐层读取git配置时，同时检查文件所有权人，如果非本用户，则停止。如果想添加例外，则需要使用上面提到的 safe.directory。
