---
title: Hello World
date: 2021-08-11 14:25:25
tags: Hexo
---
## 1. Hexo Quick Start

Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)

# 其它

## 1. 图床

- oss + PicGo

  > - Gitee中访问大于1M的图片，需要登录。对于较大的，如git，需要压缩。
  >
  >   > imagemagick
  >   >
  >   > ```shell
  >   > $ brew install imagemagick
  >   > # -fuzz 代表压缩率
  >   > $ convert pattern_design.gif -fuzz 15% -layers Optimize new_pattern_design.gif
  >   > ```
  >   
  > - Gitee中不允许有过多外链，不允许作为图床使用，选用oss作为图床。

## 2. npm

- npm 使用 [npmmirror 中国镜像站](https://npmmirror.com)，使用cnpm命令替代npm命令。
