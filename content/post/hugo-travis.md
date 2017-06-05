+++
date = "2017-06-05T10:24:04+08:00"
title = "hugo travis github 自动化"
categories = ["blog"]
tags = ["hugo","travis"]

+++

> hugo 是 go 编写的 blog生成器
> 比 ruby 的 jekyll 和 nodejs 的 hexo 生成速度快

## 安装hugo

> go开发，二进制，可直接下载对应版本的[releases](https://github.com/spf13/hugo/releases)放到系统PATH
> 或包关联安装 brew,apt,yaourt

```bash
wget https://github.com/spf13/hugo/releases/download/v0.21/hugo_0.21_Linux-64bit.tar.gz
tar -xvf  hugo_0.21_Linux-64bit.tar.gz
sudo cp hugo /usr/local/bin
```

## 生成blog

> git管理, 挑选theme设置gitsubmodule, 根据theme编辑 *config.toml*, 否则可能报错

```bash
hugo new site blog
cd blog
git init
git check -b raw
git submodule add https://github.com/gyorb/hugo-dusk.git themes/hugo-dusk
echo 'public/' > .gitignore
```

*hugo-dusk* 的 *config.toml* example

```
baseurl = "/"
title = "My site."
copyright = "Copyright (c) 2017, all rights reserved."
canonifyurls = true
languageCode = "en-US"
paginate = 3
theme = "hugo-dusk"

googleAnalytics = ""
disqusShortname = ""

[author]
  name = ""

SectionPagesMenu = "main"

[[menu.main]]
  name = "Posts"
  weight = -120
  identifier = "post"
  url = "/post/"

[[menu.main]]
  name = "Tags"
  weight = -110
  identifier = "tag"
  url = "/tags/"

[params.meta]
  keywords = "blog, tech"
  description = "Personal blog."

[params]
  github = "github id"
  gitlab = "gitlab id"
  twitter = "twitter id"
  linkedin = "linkedin id"
  email = "myemail"
```

创建 *post* , 生成 浏览

```bash
hugo new post/test.md
hugo
hugo server -w
```

## 关联github
