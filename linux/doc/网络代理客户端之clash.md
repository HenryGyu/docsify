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

* #### 安装核心程序

**1. 在官方的 [Releases](https://github.com/Dreamacro/clash/releases) 界面，下载适合服务器的客户端**

```shell
npm install pnpm -g
git clone https://github.com/Dreamacro/clash-dashboard.git
cd clash-dashboard/
pnpm install
pnpm build
pnpm start
# This command will start Clash Dashboard at http://localhost:8080/
```

?> 这里的7890和7891需设置成你真实的代理端口

**2. 添加之后保存关闭并使配置文件生效：**

```shell
source ~/.bashrc
```

**3. 启用代理**

```shell
proxyon
```

**4. 测试代理是否生效**

```shell
curl -I google.com
```

**5. 关闭代理**

```shell
proxyoff
```

!> **小提示：如果在sh文件中使用了环境变量，不能直接使用命令`sh xxx.sh`执行脚本，这样会不生效，需要使用命令`source xxx.sh`**
