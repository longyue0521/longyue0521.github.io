---
title: CentOS7 常见问题
date: 2019-01-03 13:10:45
lastmod: 2019-01-03 11:10:45

categories: ["Linux", "CentOS7", "FAQ"]
tags: ["CentOS7", "FAQ"]

draft: false
autoCollapseToc: false
mathjax: false
keywords: []
description: ""
---

{{% admonition success %}}
CentOS7使用过程中遇到的坑
{{% /admonition %}}

<!--more-->

## 查看版本

```shell
$ uname -a
# Linux host.localdomain 4.17.5-1.el7.elrepo.x86_64 \
#1 SMP Sun Jul 8 10:40:01 EDT 2018 x86_64 x86_64 x86_64 GNU/Linux

$ cat /etc/redhat-release
# CentOS Linux release 7.5.1804 (Core)

$ rpm -q centos-release
# centos-release-7-5.1804.el7.centos.2.x86_64
```

- https://www.cnblogs.com/woshimrf/p/centos-version.html

## 添加环境变量

### 添加路径

```
# 编辑~/.bash_profile或~/.bashrc文件
# 加入如下内容
export PATH=<your path 1>:<your path 2>:$PATH
```

### 立即生效

```shell
$ source ~/.bashrc (or ~/.bash_profile)
```

## Shell提示符高亮

- https://blog.csdn.net/co_zy/article/details/71464544
- https://www.cnblogs.com/lienhua34/p/5018119.html

## 找不到Shasum

```shell
$ sudo yum install -y perl-Digest-SHA
```