## GTK 主题 / 图标主题 / Shell 主题 / 声音主题 / 光标主题

主题喜好纯粹是个人口味问题，这里提供一个搜索链接来帮助你找到各类主题。当然，保持默认也是一个好选择。

* [GTK 主题](https://archlinux.org/packages/?sort=&q=gtk-theme&maintainer=&flagged=)
* [图标主题](https://archlinux.org/packages/?sort=&q=icon-theme&maintainer=&flagged=)
* Shell 主题：一般随 GTK 主题附带安装
* [声音主题](https://archlinux.org/packages/?sort=&q=sound-theme&maintainer=&flagged=)

> “愿你美化半生，归来仍是默认”。这里的默认指的是英文中的“Vanilla”，也就是香草（香子兰）。之后我会翻译中文维基百科上的“香草软件”条目，让读者更方便地了解计算机术语中的“Vanilla”。

## 顶栏透明

GNOME Shell 设计时就考虑到自定义性，支持通过 CSS 来调整 GNOME Shell 的显示方式。

首先，确保已经安装了“扩展”和“优化”两个工具。（Manjaro GNOME 应该自带这两个工具），如果没有安装，按照下面的命令安装：

```
sudo pacman -S gnome-extensions-app gnome-tweaks 
```

打开“扩展”，里面应该已经有了一些基本的扩展，启用其中的``` User Themes ```。

然后打开“优化”，进入外观版块，查看自己目前所用的 Shell 主题，然后在文件管理器中定位到```/usr/share/themes```，找到该主题的名字（如```Pop```），将该主题文件夹复制一份并改名（比如原来是```Pop```则修改为```Pop-Modified```）。

> 理论上直接在原主题文件夹修改也是可以的，但是如果这样做，每次当主题更新时就会抹去你对主题文件做出的更改，所以不建议。

进入你改名的文件夹（比如```Pop-Modified```）中的```gnome-shell```目录，编辑其中的```gnome-shell.css```。

首先通过```Ctrl+F```定位到该文件```#panel``` 部分，然后找到其中是否有 ```background-color```相关的项目。

如果有的话直接将该行修改为如下的内容，如果没有的话自行在该部分新建一行也可以。

```
  background-color: rgba(0, 0, 0, 0.3);
```

> 此步骤假定阅读者掌握基本的 CSS 语法，这里提醒阅读者在修改时不要漏掉语句最后的分号。

最后重新打开“优化”工具，进入外观版块，将 Shell 主题这项更改为改名文件夹的名称即可（比如```Pop-Modified```）。这项更改是立刻生效的。

# 安装扩展：前提和基本步骤

GNOME 扩展在 [GNOME Extensions](https://extensions.gnome.org/) 下载和安装。

从这种方式安装扩展需要安装软件包```chrome-gnome-shell```和针对浏览器安装的一个插件（这个插件会在扩展详情的页面顶部提示你安装）。除此之外最好安装``````gnome-extensions-app``````这个包，否则无法在本地调整扩展设置。

> 当然，你也可以忽略上面的步骤然后手动下载和安装扩展，只要不嫌麻烦。

## 触发角，显示桌面与“侧边鼠标滚轮调整音量”

Windows 上我常用一个软件叫做 MouseInc，但经常使用不是其特色的“鼠标手势”功能，而是“触发角”（Hot Corner）以及“侧边鼠标滚轮调整音量”的功能。

有一个扩展可以达到类似的目的，即[Custom Hot Corners Extended](https://extensions.gnome.org/extension/4167/custom-hot-corners-extended/)。它可以帮助我们自定义触发角的行为和触发角的覆盖区域。包括，如果不喜欢 GNOME 默认触发角行为的（即鼠标移到左上角时自动进入“活动”页面）也可以通过此扩展来移除。

我习惯的设置是将右上角设置为“应用程序菜单”（这个菜单可以是 Arc Menu 或者 GNOME Application Menu），右下角设置为“显示桌面”。

一个技巧是，左上角触发角的覆盖区域可以调节到“几乎整个屏幕左侧”，而触发角可选行为中包含“Sound”部分。这两个原理的结合可以变相的实现“侧边鼠标滚轮调整音量”的功能。

> 小提示：如果屏幕尺寸不大，可能无法看到完整的行为列表，此时可以善用键盘上的方向键。

## 全局菜单

[Fildem Global Menu](https://extensions.gnome.org/extension/4114/fildem-global-menu/)。

有不少 Bug，但是确实可以用。在安装和设置时需要踩许多坑。查阅 GitHub Issues 和项目 Wiki 可以解决大部分问题。

但是，只有在支持的应用程序中才可用，所以在大部分应用上不生效是正常现象。实际上并不实用。

有时会导致菜单内容失灵，比如有概率导致 Icalingua 菜单选项失效。

## 开始菜单

如果你和我一样不喜欢 GNOME 默认的应用程序屏幕，你可以考虑以下选项。

* [Arc Menu](https://extensions.gnome.org/extension/1228/arc-menu/)。其提供了非常多种样式（比如有模仿 Win11 风格的，模仿 macOS 风格的，模仿 KDE 风格的，尽管没有一个是像的）

除了样式很多，它可以被设置为放置在顶栏的左端或者右端，视乎个人习惯而定，你也可以设置其文字和图标。

* [Applications Menu](https://extensions.gnome.org/extension/6/applications-menu/)。这个应该无需安装而是自带的，功能和自定义性比上一个缺乏了许多，但是也堪用。

## 任务栏或者 Dock

这个用到的是经典的[Dash to Dock](https://extensions.gnome.org/extension/307/dash-to-dock/)。

设置为面板模式就是任务栏，默认的就是简单的 Dock。可以设置透明度，外观样式，图标大小等等。

这里建议在其设置中修改“行为”-“点击图标”，改为“最小化”，更加符合 Windows 的习惯。

除此之外还有```docky```和```cairo-dock```可以作为dock使用。

## 信息显示

* 天气显示：[OpenWeather](https://extensions.gnome.org/extension/750/openweather/)。国内使用可能速度欠佳。
* 网速显示：[NetSpeed](https://extensions.gnome.org/extension/104/netspeed/)。

## 修复托盘程序图标不显示

GNOME 从某个版本起取消了托盘功能。我们可以用[AppIndicator and KStatusNotifierItem Support](https://extensions.gnome.org/extension/615/appindicator-support/)加回来。

类似的扩展还有：[TopIcons Fix](https://extensions.gnome.org/extension/1674/topiconsfix/)

## 桌面图标

[Desktop Icons NG](https://extensions.gnome.org/extension/2087/desktop-icons-ng-ding/)

## 其它推荐的扩展

[Unite](https://extensions.gnome.org/extension/1287/unite/)

[GNOME 4x UI Improvements](https://extensions.gnome.org/extension/4158/gnome-40-ui-improvements/)

# NVIDIA 专有驱动使用 Wayland 会话

> 问：为什么要使用 Wayland？

> 答：Wayland 是新一代的显示服务器，实现上优于传统的 X11。

建议参考[这个来自 Manjaro 论坛的帖子](https://forum.manjaro.org/t/howto-use-wayland-with-propietary-nvidia-drivers/36130)。这里简述其基本步骤。

1.修改```/etc/gdm/custom.conf```然后注释掉```WaylandEnable=false```

> 注释就是在行前加一个```#```。这一行正常来说应该已经是被注释掉的状态了。

2.执行 ```sudo ln -s /dev/null /etc/udev/rules.d/61-gdm.rules```

3.执行```gsettings set org.gnome.mutter experimental-features '["kms-modifiers"]'```

4.编辑```/etc/mkinitcpio.conf```，在```MODULES```这里加入```nvidia``` , ```nvidia_modeset``` , ```nvidia_uvm``` 和```nvidia_drm ```。

然后执行```sudo mkinitcpio -P```

编辑```/etc/default/grub```，在```GRUB_CMDLINE_LINUX_DEFAULT```这里加入```nvidia-drm.modeset=1```

然后执行```sudo update-grub```

5.执行 ```sudo pacman -Syu --needed xorg-xwayland libxcb egl-wayland```

6.重启进入Wayland会话。
