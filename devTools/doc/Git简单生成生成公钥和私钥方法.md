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

4. 设置专用的ssh key（可选）
```shell
# -b 4096: 生成长度4096位的RSA密钥，安全性更好。可选，默认2048位。
# -f ~/.ssh/id_rsa_autossh: 指定文件名，不会覆盖原来的id_rsa。
# 生成后会得到：
#   私钥：~/.ssh/id_rsa_autossh_cnbmil-wms-server
#   公钥：~/.ssh/id_rsa_autossh_cnbmil-wms-server.pub
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_autossh_cnbmil-wms-server -C "root@sskj-k8s-master1_autossh"
```

---

**免密登录及传输文件**

1. 将公钥复制到目标机器
```shell
# 将 user 替换为目标机器上的用户名，将 machine_b_ip 替换为目标机器的IP地址。
# 执行后，系统会提示输入目标机器的密码。成功后，就能够免密登录到目标机器。
ssh-copy-id user@machine_b_ip

# 如果要传输指定公钥（如果端口不是22，可以通过`-p`参数指定）
ssh-copy-id -i ~/.ssh/id_rsa_autossh_cnbmil-wms-server.pub -p 22 user@machine_b_ip
```

2. 传输文件
```shell
# 方式一：使用scp传输文件
scp /path/to/file user@machine_b_ip:/path/to/destination

# 方式二：使用rsync同步文件或文件夹
rsync -avz /path/to/folder user@machine_b_ip:/path/to/destination
```

---

**使用autossh创建SSH隧道**

1. 临时创建
```shell
# -i ~/.ssh/id_rsa_autossh_cnbmil-wms-server: 指定使用哪个密钥连接（可选）
# -p 22: 指定目标主机端口（可选）
# -f: 后台运行（如果配合systemd使用，则不需要该参数，因为已经systemd负责让进程在后台运行）
# -N: 不执行远程命令（只建立隧道）
# -T: 不分配伪终端，纯粹地做端口转发，推荐加上（可选）
# -M: 使用autossh自带的端口监控模式，后面跟上数字代表本地端口号，autossh会使用该端口号来测试连接
#    （systemd模式下，推荐使用`-M 0`，表示禁用autossh自带的端口监控，然后使用ssh自带的检测机制）
autossh -M 0 -f -N \
  -i ~/.ssh/id_rsa_autossh_cnbmil-wms-server \
  -L 13306:127.0.0.1:3306 \
  -L 16379:127.0.0.1:6379 \
  -p 22 user@machine_b_ip
```

2. 配置开机自启动文件
`vim /etc/systemd/system/autossh_cnbmil-wms-server.service`
```
[Unit]
Description=AutoSSH tunnel for cnbmil-wms-server
After=network.target

[Service]
ExecStart=/usr/bin/autossh -M 0 -N -T \
    -o "ServerAliveInterval 60" \
    -o "ServerAliveCountMax 3" \
    -i ~/.ssh/id_rsa_autossh_cnbmil-wms-server \
    -L 0.0.0.0:13306:127.0.0.1:3306 \
    -L 0.0.0.0:16379:127.0.0.1:6379 \
    -p 22 user@machine_b_ip
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

3. 设置开机自启动
```shell
sudo systemctl daemon-reload
sudo systemctl enable autossh_cnbmil-wms-server
sudo systemctl start autossh_cnbmil-wms-server
```

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

---

**仓库过大，导致Git(拉取/推送)代码失败**

处理方案：默认`git`设置`http post`的缓存为`1MB`，使用命令将`git`的缓存设为`512M`
```shell
git config --global http.postBuffer 524288000
```

