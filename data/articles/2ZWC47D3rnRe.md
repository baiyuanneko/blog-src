你听说过 NixOS 吗？

一个 Linux 发行版，但它有两大特点：

* 从**几个简单的配置文件**（```configuration.nix```）出发构建整个操作系统，建立了配置文件与操作系统的函数般的一一对应关系（其它的操作系统通常是，从操作系统出发，配置文件只是为操作系统进行定制和修改的工具）
* 完全使用 Nix 包管理器来管理软件包，从根源上杜绝了软件依赖关系之间的问题。


如果想要进一步地了解它，可以参考这篇[由 bobby285271 编写的帖文](https://www.zhihu.com/question/56543855/answer/491883533)。（我也是从这篇帖文出发入门 NixOS 的）。

## 如何安装 NixOS？

**最根本的还是 [阅读官方手册](https://nixos.org/manual/nixos/stable)，下面的步骤只是简单介绍说明。** 

然后我的建议是在安装前做好以下三件事：

* 最好有 Arch Linux 或者 AOSC Linux 或者 Gentoo Linux 的安装经验，否则你不能理解很多步骤
* 了解 NixOS 配置文件的语法
* 了解 NixOS 的常见命令及其用法（如```nixos-rebuild switch```）

---

首先是下载镜像，制作安装介质并从安装介质启动。安装镜像的选择，如果你没有特别的需求的话就应该下载 NixOS Graphical ISO Image 的 GNOME 版本。（不然你的分区，联网，看手册都会非常不方便。）

然后是联网，分区和挂载。如果你用的是Graphical ISO Image 直接在桌面环境联网就好，而分区和挂载的步骤和 Arch Linux 几乎完全一致。

然后是**换源**，步骤可以参考[这个教程](https://mirror.sjtu.edu.cn/docs/nix-channels/store)。

接下来是生成新系统的默认配置文件：
```
nixos-generate-config --root /mnt
```

建议拿你喜欢的编辑器调整一下刚刚创建的```/mnt/etc/nixos/configuration.nix```。里面有很多注释所以应该还挺好理解的。

而且你会发现这个自动生成的配置文件已经包含桌面环境的安装和启动引导器的安装，所以说这个工作会比想象中的简单。

最后执行安装命令：
```
nixos-install
```

## ```configuration.nix```怎么写？

这个文件完全决定了系统长什么样，所以修改这个文件，就是我们安装软件包，修改配置的必由之路。

除了```nixos-generate-config```默认生成的那些配置之外，至少还有这些事要做：

* 中文字体：可以参考[这个配置文件](https://github.com/bobby285271/nixos-config/blob/master/desktop/fonts.nix)
* 中文 Locale
* 中文输入法：可以参考[这个配置文件](https://github.com/bobby285271/nixos-config/blob/master/desktop/fcitx.nix)
* 安装软件包：[Nixpkgs search](https://search.nixos.org/packages)
* 进行其它配置：[NixOS Search - Options](https://search.nixos.org/options)

一些常用的软件，比如 Fcitx，官方手册也有介绍过它们的安装方式，所以安装软件前在手册中搜索一番也是很重要的。

## 运行第三方软件包

Nixpkgs 中可用的软件包对于日常使用不是很够用。运行第三方软件包是一个显然的需求。

**最根本的还是[阅读官方文档](https://nixos.org/manual/nixpkgs/stable/)** ，不过考虑到这篇文档比较复杂，下面给出一些常见的方法供参考：

* 对于 AppImage 程序，不论是运行还是打包，都需要使用 [AppImage Tools](https://nixos.org/manual/nixpkgs/stable/#sec-pkgs-appimageTools) 进行
* 对于 Flatpak 程序，直接使用 Nixpkgs 中的 ```flatpak``` 即可
* 参考 Nixpkgs 中已有的软件包构建文件，自行编写新的 Nix 软件包构建文件

## 题外话

为什么我想尝试 NixOS 呢？就我个人而言是因为觉得相比其它发行版配置文件的复杂和零散，NixOS 从另一个角度看待配置文件的意义和作用，设计出了兼具**可重复性**，**声明式**，**可靠性**的发行版，这种理念和角度令我耳目一新。

然而，这个发行版的使用人数不多，因此打包不多，不能很方便的使用许多软件，我暂且没有更深入地研究。