# 命令行设置网络代理

clash是一款开源的网络代理工具。它分为标准版和高级版，高级版目前是闭源的，但是可以免费使用。  
官方GitHub地址：[https://github.com/Dreamacro/clash](https://github.com/Dreamacro/clash)。

这里还有一些基于clash的，带界面的客户端：  
- clash_for_windows_pkg（适用于`Windows/Mac/Linux`的客户端）: [https://github.com/Fndroid/clash_for_windows_pkg](https://github.com/Fndroid/clash_for_windows_pkg)
- clash_for_windows_pkg的汉化补丁: [https://github.com/BoyceLig/Clash_Chinese_Patch](https://github.com/BoyceLig/Clash_Chinese_Patch)
  > 汉化方法：  
  解压后，自行替换下列路径中的 app.asar 文件    
  Clash for Windows\resources\app.asar
- ClashForAndroid（适用于`Android`的客户端）: [https://github.com/Kr328/ClashForAndroid](https://github.com/Kr328/ClashForAndroid)

---

### 在Linux中安装并运行clash

如果你习惯使用Linux桌面端服务器，则你可以安装上面的`clash_for_windows_pkg`。  
但如果你使用的是不带桌面的Linux服务器，则应该使用官方提供的 [clash](https://github.com/Dreamacro/clash) (`核心程序`) 和 [clash-dashboard](https://github.com/Dreamacro/clash-dashboard) (`web管控台`)。  
具体过程如下：

**1. 在官方的 [Releases](https://github.com/Dreamacro/clash/releases) 界面，下载clash核心程序**

**2. 解压，并重命名，移动到指定位置**

```shell
# 解压
gunzip clash-linux-amd64-v1.11.4.gz
# 创建目录
mkdir -p /home/software/clash
# 重命名，并移动到指定目录下
mv clash-linux-amd64-v1.11.4 /home/software/clash/clash
# 授权
chmod -R +x /home/software/clash
```

**3. 下载并编译clash-dashboard**

```shell
# 进入任意目录
cd /home/software
# 安装pnpm（需先安装node，这里不做介绍）
npm install pnpm -g
# 克隆官方 clash-dashboard 仓库
git clone https://github.com/Dreamacro/clash-dashboard.git
# 进入仓库目录
cd clash-dashboard
# 安装依赖
pnpm install
# 编译打包
pnpm build
# 将编译后的文件，移动到指定目录下
mv dist/ /home/software/clash/ui/
```

**4. 使用命令 `vim /home/software/clash/config.yaml` 编辑配置文件。配置如下：**

```yaml
# HTTP(S) 代理端口
port: 7890

# SOCKS5 代理端口
socks-port: 7891

# Linux 和 macOS 的 redir 代理端口
# redir-port: 7892

# HTTP(S)和SOCKS4(A)/SOCKS5 混合代理端口
# mixed-port: 7890

# 本地SOCKS5/HTTP(S)服务器的身份验证
# authentication:
#  - "user1:pass1"
#  - "user2:pass2"

# 设置为true以允许从局域网的其他IP连接到本地终端服务器
allow-lan: true

# 绑定地址。这仅适用于 allow-lan 为 true 时。
# “*”：绑定所有IP地址
# 192.168.122.11：绑定单个IPv4地址
# “[aaaa:：a8aa:ff:fe09:57d8]”：绑定单个IPv6地址
bind-address: "*"

# 规则模式：Rule（规则） / Global（全局代理）/ Direct（全局直连）
mode: rule

# 设置日志输出级别 (默认级别：silent，即不输出任何内容，以避免因日志内容过大而导致程序内存溢出）。
# 5 个级别：silent / info / warning / error / debug。级别越高日志输出量越大，越倾向于调试，若需要请自行开启。
log-level: silent

# Clash 的 RESTful API
external-controller: "0.0.0.0:9090"

# 配置目录的相对路径 或 放置一些静态web资源的目录。 Clash 将在“http://{external-controller}}/ui”中提供它。
# external-ui: ui

# RESTful API 的口令（可选）
# secret: ""

profile:
  # 将web界面上 “选择” 的结果存储在 cache.db 中
  # 如果不希望出现这种行为，请设置为false
  # 如果配置文件和cache.db中，存在相同的配置项，则程序重启的时候，配置文件的该配置项，会覆盖cache.db中的
  store-selected: true
  # 存储虚假IP
  store-fake-ip: true
```

> 这里列出的都是关键配置，还需要增加你的代理服务商配置（懂的都懂）。  
  更多配置项参考：[https://github.com/Dreamacro/clash/wiki/configuration](https://github.com/Dreamacro/clash/wiki/configuration)

**5. 运行**

```shell
cd /home/software/clash
./clash -d .
```

> 命令中的 `-d` 后面是配置目录地址。上面这条命令表示配置目录为当前目录（/home/software/clash）。  
  首次运行，会自动创建 `config.yaml` 文件（如果该文件不存在），并自动下载 `Country.mmdb`。  
  因访问的是国外网站，下载 `Country.mmdb` 可能会很慢，或这出错。可提前在本地下载好，然后上传。  
  下载地址: [https://github.com/Dreamacro/maxmind-geoip/releases/latest/download/Country.mmdb](https://github.com/Dreamacro/maxmind-geoip/releases/latest/download/Country.mmdb)  

!> 到这里就已经结束了。此时，可以通过上面配置的 `clash-dashboard` 地址，连接并管理clash。  
但因为上面的配置缺少具体的代理服务商配置，需自行补全并重新运行即可。

---

### 设置clash开机自启动

**1. 使用命令 `vim /etc/systemd/system/clash.service` 编辑服务文件，内容如下：**

```
[Unit]
Description=clash
After=network-online.target
Wants=network-online.target systemd-networkd-wait-online.service

[Service]
ExecStart=/home/software/clash/clash -d /home/software/clash/

[Install]
WantedBy=multi-user.target
```

**2. 授权**

```shell
sudo chmod 664 /etc/systemd/system/clash.service
```

**3. 重新加载系统的systemd服务文件，并将我们添加的服务设置为开机自启动**

```shell
sudo systemctl daemon-reload
sudo systemctl enable clash.service
```
