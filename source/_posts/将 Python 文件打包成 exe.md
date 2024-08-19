---
title: 将 Python 文件打包成 exe
tags: 
  - Python
date: 2024-03-26 19:13:25
categories: 
  - 教程
---

## 一、利用Anaconda创建并激活虚拟环境

打开Anaconda Prompt

```shell
# 创建虚拟环境
conda create -n 虚拟环境名字 python==3.6

# 激活虚拟环境
conda activate 虚拟环境名字

# 查看当前虚拟环境已安装库
conda list
```

## 二、安装所需的库

根据代码需求安装所有用到的库，pyinstaller必须安装

```shell
pip install 库名

# 使用清华镜像源
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple 库名
```

## 三、打包

打开终端，移动到py文件所处的目录

```shell
pyinstaller -F -W your_script.py
```

- -F：生成单个文件
- -W：不显示终端