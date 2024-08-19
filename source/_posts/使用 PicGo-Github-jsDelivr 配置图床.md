---
title: 使用 PicGo-Github-jsDelivr 配置图床
tags: 
  - PicGo
  - 图床
date: 2024-01-05
categories: 
  - 教程
---

## 一、创建图床仓库

`Public` `Add a README file`勾选，其余默认
## 二、获取token

Settings/Developer settings/Personal access tokens/Tokens(classic)->Generate new token
## 三、PicGo配置
### 1. 安装PicGo

[PicGo下载地址](https://github.com/Molunerfinn/PicGo/tree/dev)
### 2. 安装GitHub Plus图床插件

插件设置->github-plus
### 3. 配置GitHub图床

图床设置/GitHub

| 图床配置名 | 任意 |
| ---- | ---- |
| 设定仓库名 | 账号名/仓库名 |
| 设定分支名 | main |
| 设定Token | 前面获取的token |
| 设定存储路径 | 不用填 |
| 设定自定义域名 | https://cdn.jsdelivr.net/gh/账户名/仓库名 |
