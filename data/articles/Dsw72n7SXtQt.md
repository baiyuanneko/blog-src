
## 什么是设备限制

根据微软官网的说法，即使电脑可以运行 Windows 10，如果不满足以下条件，也无法运行 Windows 11。

* **TPM 2.0**
* 64 位 CPU，**且 CPU 型号处于[这份名单](http://aka.ms/CPUlist)上**
* **Secure Boot** 与 UEFI

但是实际上，上述要求的粗体部分并非必须。首先，作为可信计算芯片的 TPM 模块的作用主要是为了存储 BitLocker 密钥，并进行对应数据的加密和解密。如果不启用 BitLocker，TPM 就并非必须（实际上即使启用 BitLocker，TPM 也非必选项，而是可以用自己设置的密钥进行代替）。其次，CPU 是否与操作系统兼容，应该由 CPU 是否支持某些指令集，或者是否支持某些特性来决定，而不是通过白名单决定。最后，作为引导方式的 UEFI 虽然必须，但是 Secure Boot 应该是可选的。（实际上就我个人而言是直接关闭 Secure Boot 的，因为它会给一些没有得到签名的操作系统的运行造成麻烦）。

非粗体部分是实际限制，不能用任何方式绕过。

## 设备限制的体现

* 通过安装介质安装时，在选择 Windows 版本之后安装程序会检查计算机配置是否满足最低要求，如果不满足，会弹出提示并阻止下一步操作（分区）
* Windows 预览体验通道在设置时会对计算机配置进行检测。对于 Windows 10 用户，如果不满足配置则只有 Release Preview 通道可用；对于 Windows 11 用户，如果不满足配置则无法加入任何一个通道
* Windows Update 更新时会弹出不满足要求的安装提示

## 设备限制的解决

### 适用范围

截至 2022 年 3 月 31 日，从 Windows 11 正式版到 Windows 11 22H2 Preview (Build 22581)。

### 使安装程序不检测 TPM 和 Secure Boot 状态

此方法适用于绕过通过安装介质全新安装 Windows 11 时会遇到的设备限制。

1. 按下 ```Shift+F10``` 以打开命令提示符
2. 输入```regedit```回车以打开注册表编辑器
3. 在注册表编辑器中定位到```HKEY_LOCAL_MACHINE\SYSTEM\Setup```，创建一个名为```LabConfig```的项，然后在```LabConfig```下创建两个名字分别为```BypassTPMCheck``` 和```BypassSecureBootCheck```的 ```DWORD``` 值，然后给这两个值赋值为```1```（双击键名即可打开赋值窗口）。
4. 完成后关闭注册表编辑器和命令提示符，安装程序已经不会再进行相关检测了。

### 加入以及管理预览体验计划

OfflineInsiderEnroll 是开源软件，可以自由管理预览体验计划的配置（包括是否加入预览体验计划，以及加入的通道类型）

项目下载地址：```https://github.com/abbodi1406/offlineinsiderenroll/releases/```


### 替换 AppRaiserRes 库文件

适用于解决 Windows Update 更新时弹出的不满足要求的安装提示。

对于 Build 22509 之前的版本，直接删除``` C:\$WINDOWS.~BT\Sources\appraiserres.dll```即可。

对于 Build 22509 及之后的版本（包括 22581），安装程序对于```appraiserres.dll```提出更高的要求。简单删除```appraiserres.dll```已经无法欺骗安装程序，需要继续按照如下步骤操作。

1. 在资源管理器中进入``` C:\$WINDOWS.~BT\Sources``` 目录
2. 找到```appraiserres.dll```先将其删除
3. 在[这里](https://www.dllme.com/dll/files/appraiserres_dll.html)下载Win10版的```appraiserres.dll```（注意下载时会要求你选择dll版本，版本号选择一个Win10的版本即可。因为Win10版本的这个dll不带有检查tpm2.0和cpu限制的功能，而Win11版本的这个dll有相关限制）
4. 下载完成后将下载的```appraiserres.dll```文件放入``` C:\$WINDOWS.~BT\Sources``` 。
5. 操作完成后关闭安装程序，然后通过设置打开 Windows 更新会提示“安装更新时遇到问题”，这次再次“解决问题”让其尝试更新就没有问题了。