# 整体流程
图片


所需工具
git
go
hugo


# 本地工作区
利用hugo创建、管理本地工作区
​

创建站点
```bash
$ hugo new site workspace
```
添加主题
```bash
$ cd workspace
$ git clone https://github.com/olOwOlo/hugo-theme-even themes/even
```
拷贝配置
```bash
# 当前目录为workspace
$ cp themes/even/exampleSite/config.toml .
```
添加文章
```bash
# 当前目录为workspace
$ hugo new post/my-first-post.md
```
修改文章
```bash
# 当前目录为workspace
$ vim content/post/my-first-post.md
```
```
---
title: "My First Post"
date: 2019-03-26T08:47:11+01:00
draft: true
---
This is my first post!
```
生成文件
```bash
# 当前目录为workspace
$ hugo server -D
```
查看站点
```bash
http://localhost:1313/
```
[
](http://localhost:1313/)


# 创建远程工作库


创建两个仓库，一个私有仓库github.com/longyue0521/workspace和一个公有仓库longyue0521.github.io


- workspace 包含全部源码文件，
   - 从本地workspace触发发布流程
      - 生成静态文件到public文件
      - 将public中内容通过github action推送到longyue0521.github.io
- longyue0521.github.io

​

《图片 remote-workspace》
​

# 链接远程工作库
使用git将本地工作区变为本地工作库
```bash
$ cd workspace
$ git init
```
将主题重新以git子模块的方式添加
```bash
$ rm -rf themes/even
$ git submodule add https://github.com/olOwOlo/hugo-theme-even themes/even
$ git add .
```
创建本地工作库的main分支
```bash
$ git add .
$ git commit -m "init git workspace"
```
将本地工作库与远程工作库关联
```bash
# 关联本地与远程工作库
$ git remote add origin https://github.com/longyue0521/workspace

# 设置本地库main分支追踪远程库main分支
$ git branch --set-upstream-to=origin/main main

# 查看远程工作库
$ git remote -v
```
同步本地工作库与远程工作库
```bash
# 将远程工作区内容拉到本地
$ git pull --rebase origin main

# 来自 https://github.com/longyue0521/workspace
# * branch            main       -> FETCH_HEAD
# 成功变基并更新 refs/heads/main

# 将本地worksapce repo内容推到远程
$ git push

# 或者使用下面的命令
$ git push --set-upstream origin main
```


# Github Action 自动化
​


- [https://docs.github.com/en/free-pro-team@latest/actions/learn-github-actions](https://docs.github.com/en/free-pro-team@latest/actions/learn-github-actions)
- [https://github.com/peaceiris/actions-hugo](https://github.com/peaceiris/actions-hugo)
- [https://github.com/peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)
- [
](https://github.com/marketplace/actions/github-pages-action)



[https://io-oi.me/tech/ssh-with-multiple-github-accounts/](https://io-oi.me/tech/ssh-with-multiple-github-accounts/)
[https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/connecting-to-github-with-ssh](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/connecting-to-github-with-ssh)


### 生成密钥对
```bash
$ ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
# You will get 2 files:
#   gh-pages.pub (public key)
#   gh-pages     (private key)
```
得到公钥`gh-pages.pub`和私钥`gh-pages`​
​

### 配置工作库
`gh-pages`
​

cat ~/.ssh/gh-pages
密钥（比如 id_rsa）


前往 Workspace 仓库，Settings > Secrets > New secret。


Name 必须为 GP_ACTIONS_DEPLOY_KEY。


- [https://github.com/marketplace/actions/hugo-deploy](https://github.com/marketplace/actions/hugo-deploy)
- [https://io-oi.me/tech/deploy-hugo-to-github-pages-via-github-actions/](https://io-oi.me/tech/deploy-hugo-to-github-pages-via-github-actions/)



​

### 配置博客库


`gh-pages.pub`


公钥
cat ~/.ssh/gh-pages.pub
前往 GitHub Pages 仓库，Settings > Deploy keys > Add deploy key。


务必勾选 Allow write access。


- [https://docs.github.com/en/free-pro-team@latest/developers/overview/managing-deploy-keys#setup-2](https://docs.github.com/en/free-pro-team@latest/developers/overview/managing-deploy-keys#setup-2)
- [https://gist.github.com/zhujunsan/a0becf82ade50ed06115](https://gist.github.com/zhujunsan/a0becf82ade50ed06115)
- [https://gist.github.com/holmberd/dbeb8789742acfd791747772104160fe](https://gist.github.com/holmberd/dbeb8789742acfd791747772104160fe)

[
](https://gist.github.com/holmberd/dbeb8789742acfd791747772104160fe)


### 新建配置文件


```shell
$ cd workspace
$ mkdir -p .github/workflows
$ vim .github/workflows/gh-pages.yml
```


添加如下内容：
```yaml
# Workflow to build and deploy site to Github Pages using Hugo

# Name of Workflow
name: Github Pages Deploy

# Controls when the action will run. Triggers the workflow on main request
# events but only for the master branch
on:
  push:
    branches:
      - main

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "deploy"
  deploy:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:

    # Step 1 - Checks-out your repository under $GITHUB_WORKSPACE
    - name: Checkout
      uses: actions/checkout@v2
      with:
          submodules: true  # Fetch Hugo themes
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

    # Step 2 - Sets up the latest version of Hugo
    - name: Setup Hugo
      uses: peaceiris/actions-hugo@v2
      with:
          hugo-version: 'latest'
          extended: true

    # Step 3 - Clean and don't fail
    - name: Clean public directory
      run: rm -rf public

    # Step 4 - Builds the site using the latest version of Hugo
    # Also specifies the theme we want to use
    - name: Builds site
      run: hugo --minify

    # Step 5 - Create README file
    - name: Create README file
     	run: cp gojustfor.fun.md public/README.md

    # Step 6 - Push our generated site to our gh-pages branch
    # - name: Deploy
    #   uses: peaceiris/actions-gh-pages@v3
    #   with:
    #       deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
    #       publish_branch: your-branch
    #       publish_dir: .
    #       cname: gojustfor.fun

    # Step 6 - Push our generated site to Github Pages Repo
    - name: Deploy it!
      uses: peaceiris/actions-gh-pages@v3
      with:
        deploy_key: ${{ secrets.GP_ACTIONS_DEPLOY_KEY }}
        external_repository: longyue0521/longyue0521.github.io
        publish_branch: main  # default: gh-pages
        publish_dir: ./public
        #cname: gojustfor.fun
```
重点关注第6步
```yaml
- name: Deploy it!
      uses: peaceiris/actions-gh-pages@v3
      with:
        deploy_key: ${{ secrets.GP_ACTIONS_DEPLOY_KEY }}
        external_repository: longyue0521/longyue0521.github.io
        publish_branch: main  # default: gh-pages
        publish_dir: ./public
        # cname: gojustfor.fun
```
需要我们创建`GP_ACTIONS_DEPLOY_KEY`以及Github Pages库。
[https://github.com/marketplace/actions/github-pages-action](https://github.com/marketplace/actions/github-pages-action)
​

​

# 自定义站点配置
```yaml
baseURL = "https://longyue0521.github.io"
languageCode = "zh-cn"
defaultContentLanguage = "zh-cn"                             # en / zh-cn / ... (This field determines which i18n file to use)
title = "Talent Is Enduring Patience"
preserveTaxonomyNames = true
enableRobotsTXT = true
enableEmoji = true
theme = "even"
enableGitInfo = false # use git commit log to generate lastmod record # 可根据 Git 中的提交生成最近更新记录。

# Syntax highlighting by Chroma. NOTE: Don't enable `highlightInClient` and `chroma` at the same time!
pygmentsOptions = "linenos=table"
pygmentsCodefences = true
pygmentsUseClasses = true
pygmentsCodefencesGuessSyntax = true

hasCJKLanguage = true     # has chinese/japanese/korean ? # 自动检测是否包含 中文\日文\韩文
paginate = 5                                              # 首页每页显示的文章数
disqusShortname = "" # "outofmemory-me"      # disqus_shortname
googleAnalytics = ""      # UA-XXXXXXXX-X
copyright = ""            # default: author.name ↓        # 默认为下面配置的author.name ↓

[author]                  # essential                     # 必需
  name = "龍躍"

[sitemap]                 # essential                     # 必需
  changefreq = "weekly"
  priority = 0.5
  filename = "sitemap.xml"

[[menu.main]]             # config your menu              # 配置目录
  name = "首页" #"Home"
  weight = 10
  identifier = "home"
  url = "/"
[[menu.main]]
  name = "归档" #"Archives"
  weight = 40
  identifier = "archives"
  url = "/post/"
[[menu.main]]
  name = "标签" #"Tags"
  weight = 30
  identifier = "tags"
  url = "/tags/"
[[menu.main]]
  name = "分类" #"Categories"
  weight = 20
  identifier = "categories"
  url = "/categories/"

[params]
  version = "4.x"           # Used to give a friendly message when you have an incompatible update
  debug = false             # If true, load `eruda.min.js`. See https://github.com/liriliri/eruda

  since = "2017"            # Site creation time          # 站点建立时间
  # use public git repo url to link lastmod git commit, enableGitInfo should be true.
  # 指定 git 仓库地址，可以生成指向最近更新的 git commit 的链接，需要将 enableGitInfo 设置成 true.
  gitRepo = ""

  # site info (optional)                                  # 站点信息（可选，不需要的可以直接注释掉）
  logoTitle = "Go! Just for fun!"        # default: the title value    # 默认值: 上面设置的title值
  keywords = ["Go", "K8s","Docker"]
  description = "Talent Is Enduring Patience | Just For Fun"

  # paginate of archives, tags and categories             # 归档、标签、分类每页显示的文章数目，建议修改为一个较大的值
  archivePaginate = 50

  # show 'xx Posts In Total' in archive page ?            # 是否在归档页显示文章的总数
  showArchiveCount = true

  # The date format to use; for a list of valid formats, see https://gohugo.io/functions/format/
  dateFormatToUse = "2006-01-02"

  # show word count and read time ?                       # 是否显示字数统计与阅读时间
  moreMeta = true

  # Syntax highlighting by highlight.js
  highlightInClient = false

  # 一些全局开关，你也可以在每一篇内容的 front matter 中针对单篇内容关闭或开启某些功能，在 archetypes/default.md 查看更多信息。
  # Some global options, you can also close or open something in front matter for a single post, see more information from `archetypes/default.md`.
  toc = true                                                                            # 是否开启目录
  autoCollapseToc = false   # Auto expand and collapse toc                              # 目录自动展开/折叠
  fancybox = true           # see https://github.com/fancyapps/fancybox                 # 是否启用fancybox（图片可点击）

  # mathjax
  mathjax = false           # see https://www.mathjax.org/                              # 是否使用mathjax（数学公式）
  mathjaxEnableSingleDollar = false                                                     # 是否使用 $...$ 即可進行inline latex渲染
  mathjaxEnableAutoNumber = false                                                       # 是否使用公式自动编号
  mathjaxUseLocalFiles = false  # You should install mathjax in `your-site/static/lib/mathjax`

  postMetaInFooter = true   # contain author, lastMod, markdown link, license           # 包含作者，上次修改时间，markdown链接，许可信息
  linkToMarkDown = true    # Only effective when hugo will output .md files.           # 链接到markdown原始文件（仅当允许hugo生成markdown文件时有效）
  contentCopyright = '<a rel="license noopener" title="国际许可(CC BY-NC-SA 4.0)"href="https://creativecommons.org/licenses/by-nc-sa/4.0/" target="_blank">"署名-非商业性使用-相同方式共享 4.0 国际"</a> 转载请保留作者及原文链接'     # e.g. '<a rel="license noopener" href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>'

  changyanAppid = ""        # Changyan app id             # 畅言
  changyanAppkey = ""       # Changyan app key

  livereUID = ""            # LiveRe UID                  # 来必力

  baiduPush = false        # baidu push                  # 百度
  baiduAnalytics = ""      # Baidu Analytics
  baiduVerification = ""   # Baidu Verification
  googleVerification = ""  # Google Verification         # 谷歌

  # Link custom CSS and JS assets
  #   (relative to /static/css and /static/js respectively)
  customCSS = []
  customJS = []

  uglyURLs = false          # please keep same with uglyurls setting

  [params.publicCDN]        # load these files from public cdn                          # 启用公共CDN，需自行定义
    enable = true
    jquery = '<script src="https://cdn.jsdelivr.net/npm/jquery@3.2.1/dist/jquery.min.js" integrity="sha256-hwg4gsxgFZhOsEEamdOYGBf13FyQuiTwlAQgxVSNgt4=" crossorigin="anonymous"></script>'
    slideout = '<script src="https://cdn.jsdelivr.net/npm/slideout@1.0.1/dist/slideout.min.js" integrity="sha256-t+zJ/g8/KXIJMjSVQdnibt4dlaDxc9zXr/9oNPeWqdg=" crossorigin="anonymous"></script>'
    fancyboxJS = '<script src="https://cdn.jsdelivr.net/npm/@fancyapps/fancybox@3.1.20/dist/jquery.fancybox.min.js" integrity="sha256-XVLffZaxoWfGUEbdzuLi7pwaUJv1cecsQJQqGLe7axY=" crossorigin="anonymous"></script>'
    fancyboxCSS = '<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fancyapps/fancybox@3.1.20/dist/jquery.fancybox.min.css" integrity="sha256-7TyXnr2YU040zfSP+rEcz29ggW4j56/ujTPwjMzyqFY=" crossorigin="anonymous">'
    timeagoJS = '<script src="https://cdn.jsdelivr.net/npm/timeago.js@3.0.2/dist/timeago.min.js" integrity="sha256-jwCP0NAdCBloaIWTWHmW4i3snUNMHUNO+jr9rYd2iOI=" crossorigin="anonymous"></script>'
    timeagoLocalesJS = '<script src="https://cdn.jsdelivr.net/npm/timeago.js@3.0.2/dist/timeago.locales.min.js" integrity="sha256-ZwofwC1Lf/faQCzN7nZtfijVV6hSwxjQMwXL4gn9qU8=" crossorigin="anonymous"></script>'
    flowchartDiagramsJS = '<script src="https://cdn.jsdelivr.net/npm/raphael@2.2.7/raphael.min.js" integrity="sha256-67By+NpOtm9ka1R6xpUefeGOY8kWWHHRAKlvaTJ7ONI=" crossorigin="anonymous"></script> <script src="https://cdn.jsdelivr.net/npm/flowchart.js@1.8.0/release/flowchart.min.js" integrity="sha256-zNGWjubXoY6rb5MnmpBNefO0RgoVYfle9p0tvOQM+6k=" crossorigin="anonymous"></script>'
    sequenceDiagramsCSS = '<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/bramp/js-sequence-diagrams@2.0.1/dist/sequence-diagram-min.css" integrity="sha384-6QbLKJMz5dS3adWSeINZe74uSydBGFbnzaAYmp+tKyq60S7H2p6V7g1TysM5lAaF" crossorigin="anonymous">'
    sequenceDiagramsJS = '<script src="https://cdn.jsdelivr.net/npm/webfontloader@1.6.28/webfontloader.js" integrity="sha256-4O4pS1SH31ZqrSO2A/2QJTVjTPqVe+jnYgOWUVr7EEc=" crossorigin="anonymous"></script> <script src="https://cdn.jsdelivr.net/npm/snapsvg@0.5.1/dist/snap.svg-min.js" integrity="sha256-oI+elz+sIm+jpn8F/qEspKoKveTc5uKeFHNNVexe6d8=" crossorigin="anonymous"></script> <script src="https://cdn.jsdelivr.net/npm/underscore@1.8.3/underscore-min.js" integrity="sha256-obZACiHd7gkOk9iIL/pimWMTJ4W/pBsKu+oZnSeBIek=" crossorigin="anonymous"></script> <script src="https://cdn.jsdelivr.net/gh/bramp/js-sequence-diagrams@2.0.1/dist/sequence-diagram-min.js" integrity="sha384-8748Vn52gHJYJI0XEuPB2QlPVNUkJlJn9tHqKec6J3q2r9l8fvRxrgn/E5ZHV0sP" crossorigin="anonymous"></script>'

  # Display a message at the beginning of an article to warn the readers that it's content may be outdated.
  # 在文章开头显示提示信息，提醒读者文章内容可能过时。
  [params.outdatedInfoWarning]
    enable = false
    hint = 30               # Display hint if the last modified time is more than these days ago.    # 如果文章最后更新于这天数之前，显示提醒
    warn = 180              # Display warning if the last modified time is more than these days ago.    # 如果文章最后更新于这天数之前，显示警告

  [params.gitment]          # Gitment is a comment system based on GitHub issues. see https://github.com/imsun/gitment
    owner = ""              # Your GitHub ID
    repo = ""               # The repo to store comments
    clientId = ""           # Your client ID
    clientSecret = ""       # Your client secret

  [params.utterances]       # https://utteranc.es/
    owner = "longyue0521"              # Your GitHub ID
    repo = "longyue0521.github.io"               # The repo to store comments

  [params.gitalk]           # Gitalk is a comment system based on GitHub issues. see https://github.com/gitalk/gitalk
    owner = ""              # Your GitHub ID
    repo = ""               # The repo to store comments
    clientId = ""           # Your client ID
    clientSecret = ""       # Your client secret

  # Valine.
  # You can get your appid and appkey from https://leancloud.cn
  # more info please open https://valine.js.org
  [params.valine]
    enable = false
    appId = '你的appId'
    appKey = '你的appKey'
    notify = false  # mail notifier , https://github.com/xCss/Valine/wiki
    verify = false # Verification code
    avatar = 'mm'
    placeholder = '说点什么吧...'
    visitor = false

  [params.flowchartDiagrams]# see https://blog.olowolo.com/example-site/post/js-flowchart-diagrams/
    enable = false
    options = ""

  [params.sequenceDiagrams] # see https://blog.olowolo.com/example-site/post/js-sequence-diagrams/
    enable = false
    options = ""            # default: "{theme: 'simple'}"

  [params.busuanzi]         # count web traffic by busuanzi                             # 是否使用不蒜子统计站点访问量
    enable = false
    siteUV = true
    sitePV = true
    pagePV = true

  [params.reward]                                         # 文章打赏
    enable = false
    wechat = "/path/to/your/wechat-qr-code.png"           # 微信二维码
    alipay = "/path/to/your/alipay-qr-code.png"           # 支付宝二维码

  [params.social]                                         # 社交链接
    a-email = "mailto:longyueli0521@gmail.com"
    #b-stack-overflow = "http://localhost:1313"
    #c-twitter = "http://localhost:1313"
    #d-facebook = "http://localhost:1313"
    #e-linkedin = "http://localhost:1313"
    #f-google = "http://localhost:1313"
    g-github = "http://github.com/longyue0521"
    h-weibo = "https://weibo.com/longyue521"
    #i-zhihu = "http://localhost:1313"
    #j-douban = "http://localhost:1313"
    #k-pocket = "http://localhost:1313"
    #l-tumblr = "http://localhost:1313"
    #m-instagram = "http://localhost:1313"
    #n-gitlab = "http://localhost:1313"
    o-bilibili = "https://space.bilibili.com/86465982"

# See https://gohugo.io/about/hugo-and-gdpr/
[privacy]
  [privacy.googleAnalytics]
    anonymizeIP = true      # 12.214.31.144 -> 12.214.31.0
  [privacy.youtube]
    privacyEnhanced = true

# see https://gohugo.io/getting-started/configuration-markup
[markup]
  [markup.tableOfContents]
    startLevel = 1
  [markup.goldmark.renderer]
    unsafe = true

# 将下面这段配置取消注释可以使 hugo 生成 .md 文件
# Uncomment these options to make hugo output .md files.
#[mediaTypes]
#  [mediaTypes."text/plain"]
#    suffixes = ["md"]
#
#[outputFormats.MarkDown]
#  mediaType = "text/plain"
#  isPlainText = true
#  isHTML = false
#
#[outputs]
#  home = ["HTML", "RSS"]
#  page = ["HTML", "MarkDown"]
#  section = ["HTML", "RSS"]
#  taxonomy = ["HTML", "RSS"]
#  taxonomyTerm = ["HTML"]

```


# 添加utterances评论


- 在<username>.github.io库上安装 [utterances app](https://github.com/apps/utterances)
- ​[https://github.com/apps/utterances](https://github.com/apps/utterances)
- [https://blog.eric7.site/2020/01/05/comment/](https://blog.eric7.site/2020/01/05/comment/)
- [https://utteranc.es/](https://utteranc.es/)
- [https://nusr.github.io/post/2019/2019-05-04-utterances-introduce/](https://nusr.github.io/post/2019/2019-05-04-utterances-introduce/)
