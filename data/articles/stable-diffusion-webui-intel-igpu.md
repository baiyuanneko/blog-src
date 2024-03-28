
## 前言：好时代，来临力？

Stable Diffusion 刚出来的时候我就想玩，不过我笔记本电脑上并没有 Nvidia 显卡，或者更准确的说，没有任何独立显卡，所以只能使用纯 CPU 跑图，生成一张图常常需要好几分钟，非常慢。

忘记去年什么时候就已经听说了 [OpenVINO](https://en.wikipedia.org/wiki/OpenVINO) 提供一个支持核显加速的 Stable Diffusion 发行版，但是由于各种原因，一直没能运行成功，这几天终于成功运行起来了。**核显能够跑ai的好时代，终于来临力！**

当然，使用 Intel 核显运行 Stable Diffusion WebUI 存在巨大的局限性（见后文），并且即使在核显加速启动的情况下，它的速度也并不十分快（相比纯 CPU 运算快，但不如 NVIDIA 显卡），只是本文希望，对于那些因为各种原因没有 NVIDIA 显卡的环境下（比如我自己），也能体验到 AI 作图的快乐。

## Re: 从零开始的 WebUI 安装过程

我的环境是 Windows 11 + Intel Iris Xe Graphics。官方同样支持 Linux 和 macOS，同时显卡方面，理论上不太老的 Intel 核显应该都能满足要求。

### 环境安装

首先是安装 Python 3.9 环境，这里我推荐先安装 Anaconda，然后在 Anaconda 中创建 Python 3.9 虚拟环境。本文不是 Python 或 Anaconda 的安装教程，所以这部分暂且略过。以下命令用于使用 conda 创建并激活 Python 3.9 虚拟环境，并假定你已安装 Anaconda.

如果你执行以下命令的时候没有什么效果（虚拟环境没有被激活），那么可能是因为你在使用 PowerShell，需要换成 cmd.exe 再重试执行以下命令。

```sh
conda create -n sdwebui_env python=3.9
conda activate sdwebui_env
```

### 修改 PyPI 镜像源（解决 No matching distribution found for tb-nightly 问题）

默认源在国内使用会很慢，应该更换一个国内的镜像源。

这里特别提醒：此处**不要**使用清华源。否则后期安装依赖时会遇到 [stable-diffusion-webui Issue#13363](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/13363) 这个问题（ERROR: No matching distribution found for tb-nightly）。

一个可行的解决方案是换用阿里云镜像源，修改命令如下。

```
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple
```

### 安装 WebUI 和 OpenVINO Toolkit

```
git clone https://github.com/openvinotoolkit/stable-diffusion-webui.git
cd stable-diffusion-webui
pip install --pre openvino
```

* **WebUI，启动！**

```
.\webui-user.bat
```

## 开启核显加速

打开 WebUI，乍一看似乎和正常的 WebUI 没有什么两样？我对普通的 WebUI 没有兴趣！

其实现在核显加速还没有被启用，如果要在生图的时候启用核显加速，还需要在 WebUI 的 Scripts 那里选择 **Accelerate with OpenVINO**，然后在 Select a device 这里选择 GPU，并在这里的 Sampling Method 里重新选一个，因为这里的 Sampling Method 会覆盖你在 Main UI 所选择的采样方法。

然后你可以在 Prompt 中写一些简单的提示词，比如`1girl, catgirl, white hair, blue eyes`，然后点击生成看看任务管理器中核显是不是在正常工作。注意首次生成的时候会自动重新编译模型+进行预热，而这个重新编译模型和预热的过程中不一定会使用 GPU，并且这整个过程在我的机器上需要大约几分钟时间，但之后每次生成如果没有改变模型本身（具体见下文）的话就不需要再次编译模型了。

### 速度

在我的 Intel Iris Xe Graphic 环境下，以 768x512 的图像生成，DPM++ 2M Karras 作为采样器，Prompt 范围处于 [78, 155]，生成模型使用 Anything v3.0 图像模型（文件大小约 4 GB），未添加 LoRA，CFG Scale 为 4，迭代步数设定为 50 的情况下为例：

* 编译模型大约需要 110s 左右（此项仅首次生成需要）
* 预热大约需要 30s 左右（此项仅首次生成需要）
* 生成图像大约需要 87s 左右（`~1.76s/it`）

### 什么行为会触发模型重新编译？

目前我简单试了几种情况，会导致模型重新编译的情况包括：

* 修改图片生成尺寸，如：生成尺寸从 512x512 修改到 768x512
* 更换图像生成模型或添加 LoRA

不会导致模型重新编译的情况包括：

* 修改 Prompt
* 修改 Negative Prompt
* 修改迭代次数

## 生成失败了？不关 WebUI 的事哦

你可能会想之前自己所用的提示词直接复制进去然后尝试生成，然后等着你的可能是两种错误信息：

* ValueError: prompt_embeds and negative_prompt_embeds must have the same shape when passed directly, but got: prompt_embeds torch.Size([1, 154, 768]) != negative_prompt_embeds torch.Size([1, 77, 768]).

这是因为目前 OpenVINO 版的 SD WebUI 要求 **Prompt 的 token 数和 Negative Prompt 的 token 数范围一致**（比如都 ≤77 或都属于 [78, 155]）。一些会导致生成失败的情形比如如果 Prompt 有 95 个 token，而 Negative Prompt 没有填写，就会发生此错误。查看 Prompt token 数的方法是在输入框的右上角有一个数字，那就是 token 数。

就我这边的情况来说，会比较容易出现的情况是 Prompt 比 Negative Prompt 的 token 数量更多，从而导致范围不一致。我的解决方法是在原始 Negative Prompt 的后面中不断填入 `,bad hands` 从而达到使 Negative Prompt token 数量达到和 Prompt token 数范围一致的效果。

然后你可能就会遇到另一个错误了：

* TypeError: 'SymInt' object is not subscriptable

这个应该是一个 Bug，怎么解决呢，每次遇到了就直接删除 WebUI 根目录下的 cache 目录即可。

这个 Bug 的触发条件是修改了提示词之后，导致 Prompt token 数范围发生改变。所以修改提示词的时候尽量不要使 Prompt token 数范围发生改变，否则就必须删除 cache 目录让它重新编译模型，耗费很多时间。

## 结语

现在其实有很多方法可以让核显加速 Stable Diffusion 的图像生成，除了本文中提到的来自 OpenVINO 的 Stable Diffusion WebUI 构建之外，这里再推荐一下 [mzwing](https://mzwing.eu.org/) 前几天编译的[一个支持 CLBLast 的 stablediffusion.cpp 构建](https://github.com/MZWNET/actions/releases/tag/sd-master-48bcce4)，虽然最近比较忙还没有空去尝试这个版本不过应该也可以很好的利用核显的性能，从而带来更快的图像生成。感谢这些开源贡献者，让核显上跑 AI 成为了一种可能。