# blog-resource

本项目是基于 Hugo 用于生成 GitHub 静态网页

## 1. 概念

Hugo 是用 Go 语言写的静态网站生成器（Static Site Generator）。可以把 Markdown 文件转化成 HTML 文件。

## 2. 安装 Hugo 环境

1. mac 用户 使用 `brew` 命令安装 hugo 和 go 环境
```shell
brew install hugo
brew install go
```
2. 查看是否安装成功
```shell
hugo version
go version
```

## 3. 新建 GitHub 仓库

### 3.1 创建博客源仓库
1. fork 本项目
2. 命名博客源仓库
3. 勾选 public
### 3.2 创建 GitHub Page 仓库
1. 命名 GitHub Pages 仓库，这个仓库必须使用特殊的命名格式 <username.github.io>， <username> 是自己的 GitHub 的用户名。
2. 勾选 Public，设置为公开仓库。
3. 勾选添加 README 文件，这会设置 main 分支为仓库的默认主分支，这在后面提交推送博客内容时很重要。

## 4. 克隆博客源仓库到本地

## 5. 使用 Hugo 创建网站
1. 进入项目
- archetypes：存放用 hugo 命令新建的 Markdown 文件应用的 front matter 模版
- content：存放内容页面，比如「博客」、「读书笔记」等
- layouts：存放定义网站的样式，写在layouts文件下的样式会覆盖安装的主题中的 layouts文件同名的样式
- static：存放所有静态文件，如图片
- data：存放创建站点时 Hugo 使用的其他数据
- public：存放 Hugo 生成的静态网页
- themes：存放主题文件
- config.toml：网站配置文件


