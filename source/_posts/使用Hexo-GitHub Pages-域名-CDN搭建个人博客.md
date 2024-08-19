---
title: 使用 Hexo-GitHub Pages-域名-CDN 搭建个人博客
tags: 
  - Hexo
date: 2024-08-16
categories: 
  - 教程
---

## 一、简介

### 1. Hexo是什么？
[Hexo](https://hexo.io/zh-cn/) 是一个快速、简洁且高效的静态博客框架，它基于 Node.js 运行，可以将我们撰写的 Markdown 文档解析渲染成静态的 HTML 网页。
### 2. Github Pages 是什么？
- [GitHub Pages](https://docs.github.com/zh/pages/getting-started-with-github-pages/about-github-pages) 是由 GitHub 官方提供的一种免费的静态站点托管服务，让我们可以在 GitHub 仓库里托管和发布自己的静态网站页面。
- 由于网站内容是公开在网络上的，因此注意不要发布隐私和敏感内容。当然也存在一定的使用限制：网站不超过 1 GB，月流量 100 GB 以内。
- 可以利用 GitHub Actions 和 Hexo 来实现博客或静态网站的自动化部署。例如，每次你更新博客内容（如推送新的 Markdown 文件）到 GitHub 仓库时，可以触发 GitHub Actions 自动运行 Hexo 构建网站，并将生成的静态文件自动部署到 GitHub Pages。这样，你只需专注于写作，网站的生成和发布都可以完全自动化。
### 3. 为什么要使用自定义域名和 CDN 加速?
- 使用自定义域名可以拥有更多的控制权，如更改 DNS 设置等。
- CDN 可以缓存网站的静态内容（如图片、CSS、JavaScript 文件），这些内容一旦被缓存，就会在一段时间内存储在 CDN 节点上，减少原始服务器的负载。
- CDN 可提供内置的安全功能，如 DDoS 防护、SSL/TLS 加密等，保护网站免受恶意攻击。
结合自定义域名和 CDN 加速可以显著提高部署在 GitHub Pages 的博客访问速度。
## 二、环境准备

**注意：配置过程中多处需要全局代理，请自行解决网络问题。**
### 1. 安装 Node.js 和 npm
-  Hexo 是基于 Node.js 的，所以需要先安装 Node.js 和 npm。
- Windows 从 [Node.js 官网](https://nodejs.org/zh-cn)下载安装包进行安装。
安装完成后，将 Node.js 安装目录加入环境变量，在 `CMD` 通过以下命令检查是否安装成功：
```bash
node -v
npm -v
```
记录下 Node.js 的主版本，如 `v20.16.0` 的主版本为 20。
### 2. 安装 Git
Windows 从 [Git 官网](https://git-scm.com/download/win) 下载安装包进行安装。
安装完成后，打开 `Git Bash` 配置一些基本信息：
```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```
### 3. 安装 Visual Studio Code
使用 [VS Code](https://code.visualstudio.com/Download) 便于利用内置的 Git 插件实现编辑后即可上传至 GitHub 仓库的快速流程，也可使用具备相同功能的文本编辑器或 IDE 。
## 三、Github Pages 配置

进入创建仓库页面

`Repository name` 必须为`<username>.github.io`

![](https://breeze-img.oss-cn-chengdu.aliyuncs.com/img/3346627dab800bfddeca088b6cf4fbc8-1723815699.png)
*由于本人已完成创建所以显示已经存在*

在仓库中前往 **Settings** > **Pages** > **Source** 。 将 source 更改为 **GitHub Actions**，然后保存。

创建完成后记录仓库地址。
## 四、Hexo配置

### 1. 安装 Hexo
在 `CMD` 中通过以下命令安装 Hexo：
```bash
npm install hexo-cli -g
```
安装完成后，通过以下命令检查是否安装成功：
```bash
hexo -version
```
### 2. 本地创建博客
在任意磁盘空间中创建一个名为 Blog 的目录，鼠标右键 `Git Bash Here` 打开 `Git Bash`
输入以下命令进行初始化：
```bash
hexo init
```
成功后可以在 Blog 目录中看见生成了一堆目录和文件。
然后输入以下命令生成本地的 Hexo 页面：
```bash
hexo s
```
浏览器打开地址：`http://localhost:4000/`
出现以下页面则表示创建成功：

![](https://breeze-img.oss-cn-chengdu.aliyuncs.com/img/1e8434d5555255b5797ff4e208ac2d30-1723881235.png)

`Ctrl + c` 关闭服务
### 3.  上传到 GitHub 仓库
使用 `VS Code` 打开 Blog 目录，创建 `.github/workflows/pages.yml` 文件，并填入以下内容 (将 `20` 替换为前面步骤中记下的 Node.js 主版本)：
```yml
name: Pages  
  
on:  
  push:  
    branches:  
      - main # default branch  
  
jobs:  
  build:  
    runs-on: ubuntu-latest  
    steps:  
      - uses: actions/checkout@v4  
        with:  
          token: ${{ secrets.GITHUB_TOKEN }}  
          # If your repository depends on submodule, please see: https://github.com/actions/checkout  
          submodules: recursive  
      - name: Use Node.js 20  
        uses: actions/setup-node@v4  
        with:  
          # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node  
          # Ref: https://github.com/actions/setup-node#supported-version-syntax  
          node-version: "20"  
      - name: Cache NPM dependencies  
        uses: actions/cache@v4  
        with:  
          path: node_modules  
          key: ${{ runner.OS }}-npm-cache  
          restore-keys: |  
            ${{ runner.OS }}-npm-cache  
      - name: Install Dependencies  
        run: npm install  
      - name: Build  
        run: npm run build  
      - name: Upload Pages artifact  
        uses: actions/upload-pages-artifact@v3  
        with:  
          path: ./public  
  deploy:  
    needs: build  
    permissions:  
      pages: write  
      id-token: write  
    environment:  
      name: github-pages  
      url: ${{ steps.deployment.outputs.page_url }}  
    runs-on: ubuntu-latest  
    steps:  
      - name: Deploy to GitHub Pages  
        id: deployment  
        uses: actions/deploy-pages@v4
```

在 Blog 目录下打开 Git Bash，输入以下命令：
```bash
git remote add origin <远程仓库URL>
git add .
git commit -m "你的提交信息"
git push -u origin main
```
如果本地仓库分支为 master 而远程仓库分支为 main ，则需要进行相应修改。

**注意：默认情况下 `public/` 不会被上传(也不该被上传)，确保 `.gitignore` 文件中包含一行 `public/`。**

部署完成后，前往 `<username>.github.io` 查看博客网页。

### 4. 更换 Hexo 主题（可跳过）
从 [Hexo 主题](https://hexo.io/themes/) 中挑选喜欢的主题，本人选择的是 [Clean Blog](https://github.com/klugjo/hexo-theme-clean-blog)。
找一个空间输入以下命令克隆主题：
```bash
git clone https://github.com/klugjo/hexo-theme-clean-blog.git themes/clean-blog
```
复制 themes 目录覆盖掉 Blog 目录下的 themes，然后删除其中 clean-blog 目录下的 `.git` 文件。

编辑 Blog 根目录下的 `_config.yml` ，修改以下内容：
```yml
theme: clean-blog
```

关于主题的更多页面配置如标签与目录可以在 [Clean Blog](https://github.com/klugjo/hexo-theme-clean-blog) 中看到。

然后将更改后的内容通过 Git 或 VS Code 内置 Git 上传至仓库。

### 5. 发布文章
使用 VS Code 打开 Blog 目录，在 `source/_posts` 下新建或将编辑好的 md 文件粘贴至此。
在页面中显示自定义信息，需要如下的文件头：

```md
---
title: mytitle
tags: 
  - tag1
date: 2024-08-16 21:26:53
categories: 
  - cate1
---
```

编辑完成后保存上传至 Git 即可自动生成页面。

## 五、个人域名配置

**以阿里云域名为例，域名购买和激活请自行查阅**

阿里云 > 云解析DNS > 解析设置 > 添加记录

- 记录类型：A-表示指向一个 ip 地址；CNAME-表示指向另一个域名，这里选择 CNAME
- 主机记录：@-表示无前缀，如 baidu.com；www-表示 www.baidu.com
- 记录值：指向的 GitHub Pages 域名。

![](https://breeze-img.oss-cn-chengdu.aliyuncs.com/img/bc7cdb6ed999c6455cae29fbf4294ea5-1724040562.png)

配置完毕后，打开 Blog 目录下的 source 目录，新建一个名为 `CNAME` 的文件，内容为你的个人域名，如下：

![](https://breeze-img.oss-cn-chengdu.aliyuncs.com/img/6e76790e1602bd0e16ffe6de991f9f39-1724041369.png)

等待片刻即可从个人域名访问到 GitHub Pages。

## 六、CDN配置

### 1. 注册cloudflare账号

进入 https://dash.cloudflare.com/sign-up 进行账号注册或登录。

登录后出现输入域名的页面，选择免费计划。

### 2. 域名解析
打开 CMD，通过 ping GitHub Pages 域名获取 ip 地址：

```bash
ping breeze4u.github.io
```

参考前面阿里云的 DNS 配置添加解析。

![](https://breeze-img.oss-cn-chengdu.aliyuncs.com/img/c7e31bfefecc61f5b1646e35081cd2d3-1724042065.png)

### 3. 更改 DNS 服务器

进入阿里云的域名管理页面，对 DNS 服务器进行修改。

![](https://breeze-img.oss-cn-chengdu.aliyuncs.com/img/ce123a5761561a43c4cccbb9e1e40cc6-1724042436.png)

后续根据指示开启免费 ssl 证书，调整策略为 FULL。

![](https://breeze-img.oss-cn-chengdu.aliyuncs.com/img/034fb977be8060dc688ad6bc359017c7-1724042588.png)

回到 GitHub Pages 的仓库，进入 Settings > Pages，勾选 Enforce HTTPS。

![](https://breeze-img.oss-cn-chengdu.aliyuncs.com/img/1145833cea460f664564f09abf081fdf-1724042993.png)

等待片刻，即可使用 https 快速的访问你的博客网站。