# Hugo + Github + Github Action 构建静态网站


## Github Action 自动化

- https://github.com/peaceiris/actions-hugo
- https://github.com/peaceiris/actions-gh-pages

### 新建 Workflow 配置文件

```shell
$ mkdir ${workspace}/.github/workflows/gh-pages.yml
```

### 配置公钥和密钥到仓库

首先，生成新的 SSH public/private key pair：

```shell
# 修改 username 为你自己的 GitHub 用户名
$ ssh-keygen -t ed25519 -C "your_email@example.com"
# 注意：这次不要直接回车，以免覆盖之前生成的
```
然后，将生成的公钥╱私钥的内容，分别复制和添加：

公钥（比如 id_rsa.pub）

前往 GitHub Pages 仓库，Settings > Deploy keys > Add deploy key。

务必勾选 Allow write access。

- https://docs.github.com/en/free-pro-team@latest/developers/overview/managing-deploy-keys#setup-2

密钥（比如 id_rsa）

前往 Workspace 仓库，Settings > Secrets > New secret。

Name 必须为 DEPLOY_KEY。
- https://github.com/marketplace/actions/hugo-deploy
- https://io-oi.me/tech/deploy-hugo-to-github-pages-via-github-actions/