# Windows 环境与依赖安装

本教程不依赖全局 `nb` 或 `pdm`。你遇到的报错：

```text
nb : 无法将“nb”项识别为 cmdlet、函数、脚本文件或可运行程序的名称
pdm : 无法将“pdm”项识别为 cmdlet、函数、脚本文件或可运行程序的名称
```

意思是系统找不到这两个命令。`nb` 来自 `nb-cli`，`pdm` 来自 PDM 包管理器。它们都不是 Python 自带命令，没有安装或没有加入 PATH 时就会报这个错。

为避免和系统 Python 依赖冲突，推荐使用独立虚拟环境。

## 1. 检查 Python 版本

在 PowerShell 执行：

```powershell
py -0p
```

本机已经有 Python 3.10，因此后续默认使用：

```powershell
py -3.10
```

不要默认使用 Python 3.13。虽然仓库声明 `Python >= 3.10`，但 NoneBot 和相关生态在 3.10 到 3.12 上更稳。

## 2. 创建机器人项目目录

建议把机器人项目放在仓库外面，避免把运行环境、数据库和日志混进插件源码仓库。

```powershell
New-Item -ItemType Directory -Force D:\WorldModel\bot\smart_message_bot
Set-Location D:\WorldModel\bot\smart_message_bot
```

## 3. 创建并激活虚拟环境

```powershell
py -3.10 -m venv .venv
.\.venv\Scripts\Activate.ps1
python --version
```

看到类似下面输出即可：

```text
Python 3.10.x
```

如果激活时报脚本执行策略错误，先在当前 PowerShell 会话执行：

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

这个命令只影响当前 PowerShell 窗口，不会永久修改系统策略。

## 4. 升级 pip

```powershell
python -m pip install -U pip setuptools wheel
```

## 5. 安装 NoneBot 和插件依赖

在已经激活 `.venv` 的 PowerShell 里执行：

```powershell
python -m pip install "nonebot2[fastapi]" nonebot-adapter-onebot nonebot-plugin-localstore websockets
```

如果你要安装 PyPI 上发布的插件版本：

```powershell
python -m pip install nonebot-plugin-smart-message-storage
```

如果你要直接使用当前本地仓库源码，推荐可编辑安装：

```powershell
python -m pip install -e D:\WorldModel\project\nonebot_plugin_smart_message_storage
```

可编辑安装的好处是：后续如果你修改仓库源码，机器人项目会直接使用新代码。

## 6. 验证依赖安装

```powershell
python -m pip show nonebot2 nonebot-adapter-onebot nonebot-plugin-localstore nonebot-plugin-smart-message-storage
```

能看到包名、版本和安装路径，就说明安装成功。

如果使用本地可编辑安装，`nonebot-plugin-smart-message-storage` 的位置应指向这个仓库。

## 7. 处理 simkai.ttf 字体

插件的 `/查消息` 会把搜索结果渲染成图片，源码里使用了：

```python
ImageFont.truetype("simkai.ttf", size=20)
```

这表示运行目录下需要能找到 `simkai.ttf`。Windows 通常自带楷体字体：

```powershell
Test-Path C:\Windows\Fonts\simkai.ttf
```

如果返回 `True`，复制到机器人项目目录：

```powershell
Copy-Item C:\Windows\Fonts\simkai.ttf D:\WorldModel\bot\smart_message_bot\simkai.ttf
```

如果没有这个字体，可以换成其他中文字体，但当前插件源码写死了 `simkai.ttf`，最简单的办法是准备一个同名字体文件放在机器人运行目录。

## 可选：安装 nb-cli

不推荐新手一开始依赖 `nb`，但如果你以后想使用 NoneBot 官方 CLI：

```powershell
python -m pip install nb-cli
nb --version
```

如果 `nb` 仍然无法识别，说明虚拟环境没有激活，或 Scripts 目录不在 PATH。可以直接使用：

```powershell
python -m nb_cli
```

## 可选：安装 PDM

这个教程不需要 PDM。如果你确实想用：

```powershell
python -m pip install pdm
pdm --version
```

PDM 会引入另一套项目管理方式。为了降低冲突，本教程后续全部使用 `venv + pip`。

## 可选：使用 Conda

如果你更习惯 Conda，可以用 Miniconda 创建环境：

```powershell
conda create -n smart-message-bot python=3.10
conda activate smart-message-bot
python -m pip install -U pip setuptools wheel
python -m pip install "nonebot2[fastapi]" nonebot-adapter-onebot nonebot-plugin-localstore
python -m pip install -e D:\WorldModel\project\nonebot_plugin_smart_message_storage
```

当前机器上没有检测到 `conda` 命令，所以默认不走这条路线。
