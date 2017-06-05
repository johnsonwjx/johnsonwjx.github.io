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
touch Makefile #否则会报 no buildable Go source files 
git add .
git commit -m 'init blog raw'
git checkout --orphan master
git rm -rf .
git commit -m 'init deploy branch' --allow-empty

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

## 关联github 和 travis

- 关联远程github
- 生成ssh key
- 填加github io项目的 *deploy key* 给 *travis* 发布用 ,添加 travis_key 到 gitignore
- 登录 travis 加密 *ssh私钥*

```bash
git checkout raw
git remote add origin  https://github.com/****.io.git
ssh-kengen -t 'rsa' -C 'travis' # 输入travis_key
echo 'travis_key' >> .gitignore
echo 'travis_key.pub' >> .gitignore
travis login --auto
travis whoami
touch .travis.yml
travis encrypt-file travis_key --add #  --add 自动添加 解密信息到 .travis.yaml
cat travis_key.pub | pbcopy # 添加公钥 到 github deploy key，勾选 Allow write access。  linux cat travis_key.pub | xclip -selection cllipboard
```
如果报错 *repository not known to https://api.travis-ci.org/:*  进入 travis-ci.org sync account

修改 .travis.yml 填写正确信息
例如

```
language: go
branches:
  only:
    - raw
env:
  global:
    - SSH_KEY="travis_key"
    - GIT_NAME="xxx"
    - GIT_EMAIL="xxx@gmail.com"
    - SOURCE_DIR="public"
    - DEPLOY_BRANCH="raw"
    - TEMP_DIR=$(sudo mktemp -d /tmp/$REPO_NAME.XXXX)
before_install:
- openssl aes-256-cbc -K $encrypted_xxxx_key -iv $encrypted_xxxxx_iv
  -in travis_key.enc -out travis_key -d
before_script:
  - go get -u -v github.com/spf13/hugo
script:
  - git submodule init
  - hugo
after_success:
  - chmod a+x ./scripts/deploy.sh
  - ./scripts/deploy.sh
```

下载发布脚本[deploy.sh](https://raw.githubusercontent.com/johnsonwjx/johnsonwjx.github.io/raw/scripts/deploy.sh)
```bash
mkdir scripts && cd scripts
wget https://raw.githubusercontent.com/johnsonwjx/johnsonwjx.github.io/raw/scripts/deploy.sh
git checkout master
git push -u origin master
git checkout raw
git add .
git commit -m 'add deploy resources'
git push -u origin master
```
到 <https://travis-ci.org/>
启用 io工程
设置 勾选 Build only if .travis.yml is present

success ！！
