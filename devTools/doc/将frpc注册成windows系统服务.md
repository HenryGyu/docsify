# 将frpc注册成windows系统服务

### windows系统的包管理器【choco】

Chocolatey是windows平台的一款包管理器工具，可以使的咱用户安装应用程序更加快速。

[Chocolatey官网](https://chocolatey.org/)

**安装**

在windows系统上，以管理员权限打开powershell，运行以下命令，安装Chocolatey。

```shell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

命令执行完，chocolatey就安装好了。

**验证**

在powershell中运行命令choco，会返回choco的版本相关信息，说明choco已经安装完毕了。


### 将frpc注册成windows服务

1、使用choco安装nssm

```shell
choco install nssm
```

2、使用nssm将frpc注册成windows服务实现frpc随着windows开机启动而启动。  
以管理员权限运行cmd，执行下面命令：

```shell
nssm install frpc
```

3、将程序主文件和运行参数以及配置文件配好，然后点击安装服务。

  ![](../_media/Snipaste_2023-01-18_11-27-02.png ':size=100%')

4、注册成功之后，会弹出界面显示服务安装成功。注意：首次安装不会运行，可以在windows服务管理点击运行。

---

本文转自 [https://blog.csdn.net/qq_37696855/article/details/122849406](https://blog.csdn.net/qq_37696855/article/details/122849406)，如有侵权，请联系删除。
