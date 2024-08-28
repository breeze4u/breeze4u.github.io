---
title: 将 Hexo 部署在云服务器并使用 GitHub Pages 实现自动部署
tags: 
  - Hexo
date: 2024-08-28
categories: 
  - 教程
---

## 一、环境准备
### Windows
**注意：配置过程中多处需要全局代理，请自行解决网络问题。**
#### 1. 安装 Node.js 和 npm
-  Hexo 是基于 Node.js 的，所以需要先安装 Node.js 和 npm。
- Windows 从 [Node.js 官网](https://nodejs.org/zh-cn)下载安装包进行安装。
安装完成后，将 Node.js 安装目录加入环境变量，在 `CMD` 通过以下命令检查是否安装成功：
```bash
node -v
npm -v
```
记录下 Node.js 的主版本，如 `v20.16.0` 的主版本为 20。
#### 2. 安装 Git
Windows 从 [Git 官网](https://git-scm.com/download/win) 下载安装包进行安装。
安装完成后，打开 `Git Bash` 配置一些基本信息：
```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```
#### 3. 安装 Visual Studio Code
使用 [VS Code](https://code.visualstudio.com/Download) 便于利用内置的 Git 插件实现编辑后即可上传至 GitHub 仓库的快速流程，也可使用具备相同功能的文本编辑器或 IDE 。

### 云服务器
#### 1. 安装 Git

```shell
sudo apt-get install git-core
```

#### 2. 安装 Node.js

通过 NodeSource 安装

```shell
# 更新包列表
sudo apt update

# 安装最新 LTS 版本
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
# 安装最新稳定版本
curl -fsSL https://deb.nodesource.com/setup_current.x | sudo -E bash -
# 安装 20 版本（与 Windows 保持一致）
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# 安装 Node.js
sudo apt-get install -y nodejs
```

验证安装：
```shell
node -v
npm -v
```

#### 3. 安装 Hexo

```shell
npm install -g hexo-cli
```

#### 4. 创建 Hexouser 管理用户

```bash
sudo adduser hexouser

# 赋予 sudo 权限
sudo usermod -aG sudo hexouser

# 修改密码
sudo passwd hexouser

# 切换用户
su hexouser
```

#### 5. 生成密钥对
- 如果你还没有 SSH 密钥对，可以在本地生成一个。运行以下命令：

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

这会生成一对密钥（公钥和私钥），通常会存放在 ~/.ssh/ 目录下。

- 添加公钥到你的服务器：

```bash
cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys
```

- 将私钥添加到 GitHub Secrets:

打开私钥文件（通常是 `~/.ssh/id_rsa`），然后复制其内容。
将输出的内容（整个密钥，包括 `-----BEGIN RSA PRIVATE KEY-----` 和 `-----END RSA PRIVATE KEY-----`）复制到剪贴板。

- 登录到你的 GitHub 账户，并导航到要使用的仓库。
- 在仓库页面，点击右上角的 **Settings**。
- 在左侧菜单中，点击 **Secrets and variables** -> **Actions**。
- 点击 **New repository secret** 按钮。
- 在 **Name** 字段中输入 `SSH_PRIVATE_KEY`。
- 在 **Secret** 字段中粘贴之前复制的私钥内容。
- 点击 **Add secret** 保存。

#### 6. 域名配置
由于本人使用的是阿里云香港服务器，无需备案，内地服务器请自行查阅资料。

进入阿里云域名管理，绑定所属服务器即可。

## 二、Github Pages 配置

进入创建仓库页面

`Repository name` 必须为`<username>.github.io`

![](https://breeze-img.oss-cn-chengdu.aliyuncs.com/img/3346627dab800bfddeca088b6cf4fbc8-1723815699.png)
*由于本人已完成创建所以显示已经存在*

在仓库中前往 **Settings** > **Pages** > **Source** 。 将 source 更改为 **GitHub Actions**，然后保存。

创建完成后记录仓库地址。

## 三、Hexo 配置
### Windows
#### 1. 安装 Hexo
在 `CMD` 中通过以下命令安装 Hexo：
```bash
npm install hexo-cli -g
```
安装完成后，通过以下命令检查是否安装成功：
```bash
hexo -version
```
#### 2. 本地创建博客
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
#### 3.  上传到 GitHub 仓库
使用 `VS Code` 打开 Blog 目录，创建 `.github/workflows/deploy.yml` 文件，并填入以下内容 (将 `20` 替换为前面步骤中记下的 Node.js 主版本)：
```yml
name: Deploy Hexo to Server

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v1
      with:
        node-version: '20'

    - name: Install dependencies
      run: npm install

    - name: Generate static files
      run: npx hexo generate

    - name: Deploy to Server
      env:
        SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
      run: |
        mkdir -p ~/.ssh
        echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan breezev.cn >> ~/.ssh/known_hosts
        rsync -avz --delete ./public/ hexouser@breezev.cn:/home/hexouser/blog/breeze4u.github.io/public
```
修改内容：
- `node-version`
- `./public/hexouser@breezev.cn:/home/hexouser/blog/breeze4u.github.io/public`


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

#### 4. 更换 Hexo 主题（可跳过）
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

注意：出现访问较慢且布局混乱的情况，很有可能是 css 等资源在当前网络无法访问，打开 F12 查找哪些资源的 resource 需要修改，进入 Blog 目录进行全局修改。
#### 5. 发布文章
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

### 云服务器
1. 以 hexouser 用户创建 blog 目录：

```bash
mkdir ~/blog
```

2. 克隆 Hexo 仓库：

```bash
git clone https://github.com/<username>/<username>.github.io.git
```

3. 初始化:

```bash
npm install

# 生成 public 目录
hexo -g
```

记录 public 路径：如`/home/hexouser/blog/breeze4u.github.io/public`
## 四、Nginx 配置
1. 编辑 Nginx 配置文件

```
sudo vim /etc/nginx/sites-available/hexo
```

```
server {
    server_name breezev.cn;

    location / {
        root /home/hexouser/blog/breeze4u.github.io/public;
        index index.html index.htm;
    }

    # 错误页处理
    error_page 404 /404.html;
    location = /404.html {
        root /home/hexouser/blog/breeze4u.github.io/public;
    }
}
```

修改内容：
- `server_name`
- `root`

2. 激活配置

```bash
sudo ln -s /etc/nginx/sites-available/hexo /etc/nginx/sites-enabled/
```

3. 测试 Nginx 配置

```bash
sudo nginx -t
```

4. 设置 Nginx 权限

将 `www-data` 用户添加到 `hexouser` 组：

```bash
sudo usermod -aG hexouser www-data
```

更改文件和目录的组权限：

```bash
chmod -R g+rwx /home/hexouser/blog/
```

6. 重新启动 Nginx 服务

```bash
sudo systemctl restart nginx
```

## 五、配置 Https
1. 安装 Certbot

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

2. 使用 Certbot 为你的域名生成 SSL 证书

```bash
sudo certbot --nginx -d your_domain.com -d www.your_domain.com
```

3. 自动更新证书

```bash
sudo certbot renew --dry-run
```

 4. 配置 Nginx

进入 Nginx 配置文件查看是否正确配置：

```
sudo vim /etc/nginx/sites-available/hexo
```

确保域名正确

5. 重启 Nginx

```bash
sudo systemctl restart nginx
```