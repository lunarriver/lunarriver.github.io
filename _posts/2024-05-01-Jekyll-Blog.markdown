---
layout: post
title:  "基于Jekyll和GithubPages搭建Blog"
date:   2024-05-01 10:24:08 +0800
---

#### 1 安装Ruby环境

（1）下载安装包：[Ruby Downloads](https://rubyinstaller.org/downloads/)或[ruby.cn](https://rubyinstaller.cn/)。

![img](../../../assets/jekyll-blog/ruby-install-devkit-3.3.0-1-x64.png)

（2）执行安装。安装路径不要包含空格、中文。

- 将ruby可执行目录添加到环境变量。

![img](../../../assets/jekyll-blog/ruby-install-add-path.png)

- 勾选**MSYS2 development toolchain**。

![img](../../../assets/jekyll-blog/ruby-install-msys2-toolchain.png)

- 安装完成勾选自动执行ridk。

![img](../../../assets/jekyll-blog/ruby-install-ridk.png)

- 选择第3项：MSYS2 and MINGW development toolchain。

![img](../../../assets/jekyll-blog/ruby-install-msys2-mingw.png)

出现如下执行结果即表明成功。

![img](../../../assets/jekyll-blog/ruby-install-msys2-mingw-success.png)

#### 2 安装jekyll相关依赖

```
依赖安装比较耗时，且有可能出错（中止安装），但无需科学上网，多次尝试即可。
```

（1）安装bundle。

![img](../../../assets/jekyll-blog/ruby-install-gem-bundle.png)

（2）安装jekyll。

![img](../../../assets/jekyll-blog/ruby-install-gem-jekyll.png)

（3）安装github-pages。

![img](../../../assets/jekyll-blog/ruby-install-gem-github-pages.png)

#### 3 创建github.io项目

（1）于Github上创建仓库[account-name].github.io，account-name即是Github账号。

![img](../../../assets/jekyll-blog/create-repository.png)

（2）将gitHub.io项目（空项目）拉取到本地。在项目目录下执行以下命令：

```
jekyll new .
```

![img](../../../assets/jekyll-blog/create-jekyll.png)

创建成功后，项目目录如下：

![img](../../../assets/jekyll-blog/jekyll-dir.png)

（3）执行以下命令以运行jekyll（非必须步骤，只用于查验项目创建是否无误）：

```
bundle exec jekyll serve -P 5000 --watch
```

![img](../../../assets/jekyll-blog/jekyll-serve.png)

访问运行地址（127.0.0.1:5000），查看Blog页面：

![img](../../../assets/jekyll-blog/jekyll-page.png)

#### 4 将github.io项目提交至Github

（1）将项目commit并push至Github。

![img](../../../assets/jekyll-blog/github-commit.png)

（2）设置Github Pages的分支。

![img](../../../assets/jekyll-blog/set-branch.png)

设置分支后，将立即触发构建。

![img](../../../assets/jekyll-blog/building.png)

构建完成后，可访问博客，地址为：

```
https://[account-name].github.io/
```
