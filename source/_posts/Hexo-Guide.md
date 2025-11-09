---
title: Hexo Guide
abbrlink: b08947fa
date: 2025-11-07 12:52:58
tags:
- Hexo
- Github
categories: Hexo
description: 搭建Hexo博客基本步骤，以及图床和Github的配置
cover: https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/Hexo.png
swiper_index: 1
---

# 环境配置

## Git安装

https://git-scm.com/

## Node.js安装

https://nodejs.org/en/download/

## Hexo安装

```bash
# 使用 npm 安装 Hexo
npm install -g hexo-cli

# Hexo初始化
hexo init <folder>
cd <folder>
npm install
```

# Github配置

## 创建仓库

> 仓库名称必须为`用户名 + github.io`，可视化为`Public`

## git配置

```bash
# 设置用户名和邮箱
git config --global user.name "xxx"
git config --global user.email "xxx@gmail.com"
```

## SSH KEY配置

```bash
# 创建SSH KEY,将用户文件夹下的id_rsa.pub文件的内容复制到github的SSH KEYS中
ssh-keygen -t rsa -C "xxx@gmail.com"

# 测试是否成功连接
ssh -T git@github.com
```

![github ssh key](https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/github_ssh_key.png)

> 可能会出现错误`ssh: connect to host github.com port 22: Connection timed out `
>
> ```bash
> # 由于默认的 SSH 22 端口被防火墙或网络策略限制，导致无法连接到 GitHub 的服务器
> 
> # 连接改为 SSH 的 443 端口
> touch ~/.ssh/config
> 
> # config文件添加以下内容
> Host github.com
> HostName ssh.github.com
> User git
> Port 443
> PreferredAuthentications publickey
> IdentityFile ~/.ssh/id_rsa
> 
> ssh -T git@github.com
> ```

## 一键部署

```bash
# 安装hexo-deployer-git
npm install hexo-deployer-git --save
```

```yaml
# _config.yml配置
deploy:
  type: git
  repo: git@github.com:xxx/xxx.github.io.git
  branch: main
```

## hexo源文件同步github

[网友教程](https://midoli.top/2024/12/01/Hexo%E5%9C%A8%E5%A4%9A%E5%8F%B0%E7%94%B5%E8%84%91%E4%B8%8A%E6%8F%90%E4%BA%A4%E5%92%8C%E6%9B%B4%E6%96%B0/)

> `main`分支存放hexo生成的静态文件，而`source`分支设置为默认分支，存放hexo源文件

```bash
# 复制source分支下的文件，仅保留.git文件夹
git clone git@github.com:xxx/xxx.github.io.git

# 将源文件除.deploy_git 以外都复制到clone下来的文件夹中
# 删除theme主题文件夹下的.git

# 推送源文件到source分支
git add .
git commit –m add_branch
git push
```

## github图床配置

> jsdeliver节点可能存在不可用的网络问题

1. 创建github仓库，并生成token
2. 配置PicGo([PicGo github地址](https://github.com/Molunerfinn/PicGo))，设置自定义域名https://gcore.jsdelivr.net/gh/Ethylenekun/images
3. typora设置，图像 - 上传图片 - 上传服务(PicGo_app)

## NAS图床设置

[知乎教程](https://zhuanlan.zhihu.com/p/382702959)

```bash
# 因PicGO插件功能存在问题,使用命令行安装ftp上传插件
cd C:\Users\yixi7\AppData\Roaming\picgo
npm install picgo-plugin-ftp-uploader
```

> PicGO FTP上传配置

![PicGO FTP uploader config](https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/FTP%E4%B8%8A%E4%BC%A0%E8%AE%BE%E7%BD%AE.png
)

```json
{
    "NAS": {
        "url": "http://xxx:5543/",
        "path": "/{year}/{month}/{day}/{fullName}",
        "uploadPath": "/PicBed/upload/{year}/{month}/{day}/{fullName}",
        "port": 5542,
        "host": "xxx",
        "username": "xxx",
        "password": "xxx"
    }
}
```

> NAS FTP设置：由于上传的用户少，被动端口范围设置为4200-4205

![NAS FTP config](https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/image-20251109082126997.png)

> 路由设置：FTP端口转发，FTP服务端口和被动端口都要设置转发

![image-20251109083334323](https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/image-20251109083334323.png)

![image-20251109083252737](https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/image-20251109083252737.png)

# 主题设置

[Butterfly官方文档](https://butterfly.js.org/posts/21cfbf15/)

## 快速开始

```bash
# hexo 根目录运行
git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly

# 应用主题 Hexo 根目錄下的 _config.yml，把主題改為 butterfly
theme: butterfly

# 安装 pug 以及 stylus 的渲染器
npm install hexo-renderer-pug hexo-renderer-stylus --save
```

### 升级建议

在 hexo 的根目錄創建一個文件 `_config.butterfly.yml`，並把主題目錄的 `_config.yml` 內容複製到 `_config.butterfly.yml` 

# Hexo配置文件

> `_config.yml`配置文件

```bash
# Site
title: Ethylene's Blog
subtitle: '沉思'
description: '若遇见你不是奇迹，灵魂便不曾言语'
keywords: 'Python,Pandas,MySQL,Django'
author: Ethylene
language: zh-CN
timezone: 'Asia/Shanghai'
```

```bash
# URL
url: https://xxx.xxx.io/
```

# 其他配置

## 文章链接配置

[hexo-abbrlink github地址](https://github.com/ohroy/hexo-abbrlink)

```bash
npm install hexo-abbrlink --save
```

```yaml
# _config.yml配置
permalink: posts/:abbrlink.html

# abbrlink config
abbrlink:
  alg: crc32      
  rep: hex
```

## live2D配置

[hexo-helper-live2d github地址](https://github.com/EYHN/hexo-helper-live2d)

```bash
npm install --save hexo-helper-live2d

# 安装模型
npm install --save live2d-widget-model-wanko
```

```yaml
# _config.yml配置
live2d:
  enable: true
  scriptFrom: local
  pluginRootPath: live2dw/
  pluginJsPath: lib/
  pluginModelPath: assets/
  tagMode: false
  log: false
  model:
    use: live2d-widget-model-wanko
  display:
    position: left
    width: 150
    height: 300
  mobile:
    show: true
  react:
    opacity: 0.7
```

## swiper插件配置

```bash
# 安装swiper依赖
npm install hexo-butterfly-swiper --save
```

> ` _config.butterfly.yml`配置

```yaml
swiper:
  enable: true # 开关
  priority: 5 #过滤器优先权
  enable_page: all # 应用页面
  timemode: date #date/updated
  layout: # 挂载容器类型
    type: id
    name: recent-posts
    index: 0
  default_descr: 再怎么看我也不知道怎么描述它的啦！
  swiper_css: https://npm.elemecdn.com/hexo-butterfly-swiper/lib/swiper.min.css #swiper css依赖
  swiper_js: https://npm.elemecdn.com/hexo-butterfly-swiper/lib/swiper.min.js #swiper js依赖
  custom_css: https://npm.elemecdn.com/hexo-butterfly-swiper/lib/swiperstyle.css # 适配主题样式补丁
  custom_js: https://npm.elemecdn.com/hexo-butterfly-swiper/lib/swiper_init.js # swiper初始化方法
  
pjax:
  enable: true
  exclude:
```

> `post front-matter`配置

```markdown
swiper_index: 1
```

