## 前言

WSL 是在 Windows 上配置 Linux 环境的简单方法，WSL 目前有两个版本，WSL 1 和 WSL 2。关于它们的区别和联系，可以参考 [这篇微软的官方文档](https://docs.microsoft.com/zh-cn/windows/wsl/compare-versions)。简单来说：

* WSL 2 基于 Hyper-V ，支持 Docker，支持 WSLg 因而有图形界面的原生支持，大多数情况下比 WSL1 具有更好的性能。
* WSL 1 不使用 Hyper-V，不支持 Docker，没有对图形界面的原生支持，且性能也弱于 WSL2 （跨文件系统性能除外）。

参考个人需求后，我选择了 WSL1。以下教程基于 WSL1 进行。

本教程假定你已经熟悉 Windows 和 Linux 的基础知识。

## 手动安装 WSL1

建议参考：[旧版 WSL 的手动安装步骤](https://docs.microsoft.com/zh-cn/windows/wsl/install-manual)

启用相关 Windows 功能：

```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

重启电脑使配置生效。

设置 WSL 1 为默认版本：

```
wsl --set-default-version 1
```

最后在 Microsoft Store 安装 Ubuntu。安装完成后，开始菜单会出现一个Ubuntu 的应用，直接点击，会进入 Linux 安装流程，完成后会要求配置 Linux 用户名和密码，分别输入即可。

## 为 Ubuntu 更换软件源、更新系统

编辑``` /etc/apt/sources.list``` 

替换所有的 ```archive.ubuntu.com```为```mirrors.tuna.tsinghua.edu.cn``` 

然后执行 ```sudo apt update && sudo apt upgrade``` 更新系统。

## 为 Ubuntu 添加图形环境支持

因为我们使用的是 WSL1 ，没法使用 WSLg，因此需要手动配置图形环境。

首先，安装桌面组件。（耗时比较久）

```
sudo apt install ubuntu-desktop
```

然后，执行```export DISPLAY=localhost:0```以设置 X 输出端。

并在```/etc/profile```添加同样的内容：

```
export DISPLAY=localhost:0
```

然后在 Windows 下下载安装[VCXSrv](https://sourceforge.net/projects/vcxsrv/)。

安装完成后，开始菜单中会出现 XLaunch 程序，直接打开它，所有选项都按默认配置无需更改，一路下一步即可。

完成后，右下角托盘会出现 XLaunch 程序驻留。

现在可以尝试在 Ubuntu 下打开图形程序了，如```gedit```。Linux 下的图形程序也会以窗口形式打开，并和 Windows 程序具有类似的体验。

![image](https://s2.loli.net/2022/05/06/J6IzQ17XbtNpkne.png)

## 为 Ubuntu 添加中文支持

[WSL 1 存在一个 Bug 会阻碍 Character Map 文件的解析](https://stackoverflow.com/questions/58304278/how-to-fix-character-map-file-utf-8-not-found)，因此这里需要手动在Windows 的文件资源管理器中找到并进入 Linux 文件系统（通常在文件资源管理器的侧边栏中可以看到），定位到 /usr/share/i18n/charmaps/UTF-8.gz ，把这个gz 文件拷贝到 Windows 中，使用 Windows 上的解压软件解压出 UTF-8 文件（**注意不是文件夹而是文件**），然后将解压出的 UTF-8 文件再放到 /usr/share/i18n/charmaps/ 去 。

输入以下命令安装中文语言包、中文字体和中文输入法：

```
sudo apt install language-pack-zh-hans fonts-wqy-microhei dbus-x11 im-config fcitx fcitx-pinyin
```

建议编辑```/etc/locale.gen```文件然后把```zh_CN UTF-8 UTF-8```前面的注释符号去掉。（如果不执行此操作，Locale 数据会是```zh_SG UTF-8 UTF-8```，但对后续使用没有决定性的影响）

使用```sudo locale-gen```生成 Locale 数据。

编辑```/etc/profile```，在文件末尾加入：

```
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN.UTF-8
export LC_ALL=zh_CN.UTF-8
export INPUT_METHOD=fcitx
export XMODIFIERS=@im=fcitx
export GTK_IM_MODULE=fcitx
```

然后执行```source /etc/profile```。

执行```sudo apt remove ibus```删除 ibus，因为它会对 Fcitx 造成干扰。

执行```fcitx-autostart```，这样就能在WSL启动时自动启动输入法了。

重启 WSL 后中文支持和输入法生效。

![image](https://s2.loli.net/2022/05/06/8UbvGpljIKY9BPc.png)

## 开发环境配置

许多开发工具已有对 WSL 的支持，可以直接调用 WSL 环境来执行操作。如 Visual Studio Code 的 Remote - WSL 插件等等。这一方面可参考开发工具的官方文档；WSL 本身的开发环境配置与正常的 Linux 大同小异。

