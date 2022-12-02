
![image](https://s2.loli.net/2022/07/12/AqRbeBnWVETN7d5.png)

众所周知，Linux 桌面的 HiDPI 存在着种种问题。为了守护我的视力健康，我又回到了 Windows。

然而，习惯了 Fish 的交互式体验，因此我想要在 Windows 下也默认使用 Fish.

首先，[Fish Shell 官网](https://fishshell.com/) 提供了其在 Windows 下的安装方法。总的来说归为 Cygwin、WSL 和 MSYS2 三类。我使用的是 Cygwin。

## 安装 Cygwin

在 [Cygwin 官网](https://cygwin.com/install.html) 下载 Cygwin 的安装程序。

这是一个在线安装程序，安装过程会要求你选择软件源和要安装的软件包，其中软件源我选择的是 ```mirrors.aliyun.com```，速度还是挺快的，然后在软件包选择界面中搜索```fish```，双击 ```fish``` 旁边的 ```Skip``` 字样，然后你就为 ```fish``` 指定了一个默认的安装版本，```Skip```字样会变成 Fish 的安装版本。用同样的方法安装 ```python3```，因为 ```fish_config``` 和其它的一些工具都依赖 ```python3```，然后等待安装完成。

> Cygwin 默认的安装目录是```C:\cygwin64```，如果之后想要安装什么别的软件包，或者想要修改一些配置，重新运行那个在线安装程序即可。

你可以在开始菜单找到 Cygwin64 Terminal，但是目前 Fish 还没有被设定为默认 Shell，而且你应该会希望使用更现代的 Windows Terminal。

## 将 Fish 设定为 Cygwin 的默认 Shell

参考资料：[这篇 Stack Overflow 文章](https://stackoverflow.com/questions/22363210/set-default-shell-in-cygwin)

以下操作均在 Cygwin 环境中运行。

首先编辑```/etc/nsswitch.conf```，加入或修改这一行：

```
db_shell: /usr/bin/fish
```

然后执行

```sh
mkpasswd > /etc/passwd
```

最后编辑 Cygwin 安装目录下的```Cygwin.bat``` 文件，把```bash```改成```fish```即可。

## 在 Windows Terminal 中添加 Fish on Cygwin 的配置文件

首先，更新你的 Windows Terminal 到最新版本，这样Windows Termianl 才有图形化的设置界面。

然后进入 Windows Terminal 设置，在配置文件中点击“新建配置文件”。

名称图标什么的按自己喜好来就行，命令行（可执行文件）的路径是```C:\cygwin64\bin\fish.exe```（当然如果你更改了默认安装路径就不是这个）。注意不要填成安装路径下的```Cygwin.bat```，那样虽然看起来可以工作，但是“在指定目录下打开终端”的功能就会失效。

## 在 VS Code 集成终端中默认使用 Fish Shell，并为部分工作区设置例外

在 VSCode 中按 Ctrl+Shift+P打开命令面板，搜索```setting json```打开 JSON 格式的配置文件，加入下面这一段：

```json
{
    "terminal.integrated.profiles.windows": {
        "Fish on Cygwin": {
           "path": "C:\\cygwin64\\bin\\fish.exe"
        }
    },

    "terminal.integrated.defaultProfile.windows": "Fish on Cygwin"
}
```

然后你再打开 VS Code 的集成终端看到的就是 fish 了。

但是，对于 VS Code 来说情况没有这么简单，比如，如果你使用 VS Code 开发 Java 程序，你会发现在切换到 Fish 之后，由扩展插件提供的运行和调试按钮都用不了了。可见它们还是需要调用 PowerShell 的。因此，我们需要为特定的工作区设定一些例外。（注：如果有更好的方法可以绕过这个问题，欢迎你告诉我）。

在需要调用 PowerShell 的工作目录下添加```.vscode```文件夹，在其中新建```settings.json```文件（如果已经有了，就直接编辑它），加入下面这段内容：

```json
"terminal.integrated.defaultProfile.windows": "PowerShell"
```

这样，对于该工作区来说，默认调用的 Shell 就不是 Fish 而是 PowerShell 了，然后那些运行和调试按钮又可以正常使用了。而其它的工作区还是可以正常使用 Fish，这个设置只有在该工作区才会生效。

效果图（Windows Terminal）：

![image](https://s2.loli.net/2022/07/12/AqRbeBnWVETN7d5.png)

效果图（VS Code）：

![image](https://s2.loli.net/2022/07/12/zhx21BCtGSfkrwW.png)