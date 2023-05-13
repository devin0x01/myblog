---
title: "配置github pages教程"
date: 2023-05-13T14:05:25+08:00
draft: false
---
## 参考文档

[github-style theme](https://themes.gohugo.io/themes/github-style/)  
[github actions 教程](https://www.pseudoyu.com/zh/2022/05/29/deploy_your_blog_using_hugo_and_github_action/)

## 基础配置

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

Hugo will create a post with `draft: true`, change it to false in order for it to show in the website.
`hugo new post/title_of_the_post.md`

## 本地预览

> hugo server 的选项：
> -w 修改后本地服务器可以立即变化
> -D 也显示 `draft: true`的帖子

`hugo server -t github-style -D -w`

## 部署到 Github Pages

### 手动部署

生成 `hugo --theme=your_theme --baseUrl="your_server_or_domain" --buildDrafts`  
提交 `public` 目录到 `<username>.github.io` 仓库

### 使用 Github Actions 自动部署

- 在 github 账户申请具备 repo 和 workflow 权限的密钥
- 在博客原始代码仓库添加环境变量 `PERSONAL_TOKEN`
- ![image](https://raw.githubusercontent.com/devin0x01/myimages/master/github_pagesgithub_actions.png)
- 编辑 `.github/workflow/deploy.yml` 文件

  > on 表示 GitHub Action 触发条件，我设置了 push、workflow_dispatch 和 schedule 三个条件：
  > push，当这个项目仓库发生推送动作后，执行 GitHub Action
  > workflow_dispatch，可以在 GitHub 项目仓库的 Action 工具栏进行手动调用
  > schedule，定时执行 GitHub Action，主要是使用一些自动化统计 CI 来自动更新我博客的关于页面，如本周编码时间

  > jobs 表示 GitHub Action 中的任务，我们设置了一个 build 任务，runs-on 表示 GitHub Action 运行环境，我们选择了 ubuntu-latest。
  > 我们的 build 任务包含了 Checkout、Setup Hugo、Build Web 和 Deploy Web 四个主要步骤，其中 run 是执行的命令，uses 是 GitHub Action 中的一个插件，我们使用了 peaceiris/actions-hugo@v2 和 peaceiris/actions-gh-pages@v3 这两个插件。其中 Checkout 步骤中 with 中配置 submodules 值为 true 可以同步博客源仓库的子模块，即我们的主题模块。
  > EXTERNAL_REPOSITORY 需要改为自己的 GitHub Pages 仓库

