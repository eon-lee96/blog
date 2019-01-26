+++
title = "Hugo结合travis-ci搭博客"
date = 2019-01-26T20:27:58+08:00
tags = ["其他"]
draft = false
description = "Hugo结合travis-ci搭博客"
+++

## 前言

博客引擎来来去去换了很多个，从最开始[wordpress](https://wordpress.org/)，到[hexo](https://hexo.io/zh-cn/)、[cmsjs](https://chris-diana.com/cms.js/)。最开始都是在自己的墙外服务器搭，但实际上也没怎么写文章，而且小水管也用着不爽。

为了加强自己写博客的欲望，现在改用[hugo](https://gohugo.io/)了，好吧我承认跟写博客没关系，只是想折腾一下。除了用[hugo](https://gohugo.io/)之外还打算同步维护一个[repo](https://github.com/leeEhang/blog)来写`issue-blog`。

记录一下`hugo`结合`travis-ci`部署到`GitHub-pages`的过程。

## Hugo

#### 安装

具体系统安装方式参考[官方文档](https://gohugo.io/getting-started/installing/)。

我用的是mac就直接用[homebrew](https://brew.sh/)，来安装

```bash
brew install hugo
```

#### 创建站点

直接执行如下命令创建一个站点文件夹：

```sh
hugo new site blog
```

正常来说会出现下面一段：

```bash
Congratulations! Your new Hugo site is created in 你的路径.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/, or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.
```

就是告诉你还得下载个主题才能跑起来，到[hugo主题站](https://themes.gohugo.io/)下载个主题安装就好。

#### 安装主题

有两种方式安装主题，一种是直接把仓库克隆下来当做博客仓库的一部分，一种是利用[git-Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)，后面一种目前知识浅薄不太熟悉就没用了，我就直接进入前面说的创建的`blog`站点目录下的`themes`目录下把我想要的主题clone下来。然后删掉主题内的`.git`文件夹。

主题弄好后直接

```bash
hugo server
```

就可以把博客跑起来了。

当然跑起来后是没有内容的，具体创建文章的方法需要结合主题参考[文档](https://gohugo.io/commands/hugo_new/)。

[hugo](https://gohugo.io)的使用方式具体参考官方文档就好了。

当时有一点需要注意的就是图片目录，可以把图片放在`static`下，然后在`content`里面的`markdown`内就可以直接把`static`当做根目录来寻址

## github

### travis.yml

弄好博客后就要上传到`github`了，但是我们得先配置一下`travis-ci`配置文件。

创建一个`.travis.yml`

```yaml
sudo: false
language: go
git:
    depth: 1
# 安装hugo
install: go get -v github.com/gohugoio/hugo
# 执行脚本,hugo构建站点
script:
    - hugo

# 部署配置
deploy:
  # 使用gitbuh-pages
  provider: pages
  skip-cleanup: true
  # 部署的文件夹，这里是hugo构建的public文件夹
  local-dir: public
  # github开发者TOKEN，得有这个travis才能操作repo
  github-token: $GITHUB_TOKEN
  keep-history: true
  # 部署目标分支，把产物部署到仓库的具体某个分支
  target-branch: gh-pages
  # github-pages的自定义域名，会自动创建CNAME文件
  fqdn: blog.lee1hang.com
  on:
  	# 在仓库的哪个分支上构建，默认是master，如果设置了请注意把hugo站点传到定义的分之下
    branch: page

```

### TOKEN获取

在个人的`github`设置里面找到`developer settings`

![WX20190126-194657@2x](/images/hugo4.png)

然后在`personal access tonkes`里点击`generate new token`

![WX20190126-194810@2x](/images/hugo3.png)

在出来的页面填写描述后，最少需要勾选`public_repo`的权限。

![WX20190126-195200@2x](/images/hugo2.png)

之后可以创建了`token`,需要注意的是要妥善保存`token`，因为创建成功后的展示`token`的页面只会出现一次。

## travis-ci

首先使用`github`账号登录到[travis-ci](https://travis-ci.org/)，同步账户信息后就可以在面板上看到仓库列表。选择之前我们创建的仓库进入设置面板。

![WX20190126-201803@2x](/images/hugo1.png)

注意填写之前保存的`token`，`NAME`要跟之前在`travis.yml`内定义的参数名一致（注意不要写上`$`），方便起见也可以勾选`display value in build log`。

之后我们可以触发一下`trigger build`看能不能正常构建。

正常情况下，是可以的，然后整个过程就结束了。
