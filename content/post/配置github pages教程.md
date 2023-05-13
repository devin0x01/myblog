---
title: "配置github pages教程"
date: 2023-05-13T14:05:25+08:00
draft: false
---

## 基础配置
[github-style theme](https://themes.gohugo.io/themes/github-style/)
```
mkdir myblog && cd myblog
hugo new site .

git init
git submodule add git@github.com:MeiK2333/github-style.git themes/github-style
vim .gitignore
git add .
git push -u origin master
```

## 更新主题
```
cd themes/github-style
git pull
```

## 编辑readme
```
hugo new readme.md
echo '`Hello World!`' > content/readme.md
```

## 新增帖子
Hugo will create a post with `draft: true`, change it to false in order for it to show in the website. \
`hugo new post/title_of_the_post.md`

## 预览
> hugo server 的选项：\
-w 修改后本地服务器可以立即变化 \
-D 也显示`draft: true`的帖子

`hugo server -t github-style -D -w`

## 部署到 Github Pages
1.手动部署： \
生成 `hugo --theme=your_theme --baseUrl="your_server_or_domain" --buildDrafts` \
提交public目录到 <username>.github.io 仓库

2.使用 Github Actions 自动部署: \
https://www.pseudoyu.com/zh/2022/05/29/deploy_your_blog_using_hugo_and_github_action/
