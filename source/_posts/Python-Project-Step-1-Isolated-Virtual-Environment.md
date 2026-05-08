---
title: 'Python Project Step 1: Isolated Virtual Environment'
date: 2026-05-05 16:29:41
tags: [Python Project]
categories: [自学记录]
---

- 当我们开始做项目的时侯，发现总是有各种错误，比如：缺少包，包版本不匹配，包冲突等等，那么，我们如何解决呢？
- 我们可以创建一个虚拟环境，然后安装需要的包，这样，我们就可以避免环境冲突，并且可以保证项目的运行环境是固定的，不会因为环境改变而导致项目运行错误。
- 当然，创建的虚拟环境，是使用 Python 的 venv 模块创建的，所以，我们首先需要安装 venv 模块（python3.3+版本内置不需要额外安装）。

<!-- more -->

# 学习创建虚拟环境

```bash
# 1. 进入项目文件夹，打开终端
# 2. 创建虚拟环境（会在当前目录下生成一个 venv 文件夹）
python -m venv .venv   # 第二个venv 是虚拟环境文件夹名，可自定义但一般为.venv
# 3. 激活虚拟环境（Windows）
# 如果提示权限问题，先执行 
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

.venv\Scripts\activate
# 或者 Mac/Linux
source venv/bin/activate

# 4. 激活后，安装一个包试试
pip install requests

# 5. 查看当前环境下的包
pip list

# 6. 退出虚拟环境
deactivate

# Windows PowerShell   脚本文件：Activate.ps1	
# Windows Cmd          脚本文件：activate.bat
# Mac / Linux          脚本文件：activate

```

