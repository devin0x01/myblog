---
title: "配置github pages教程"
date: 2023-05-13T14:05:25+08:00
tags: ["工具"]
categories: []
draft: false
toc: true
---

## 1.基础配置
```
mkdir myblog && cd myblog
hugo new site .

git init
git submodule add git@github.com:MeiK2333/github-style.git themes/github-style
vim .gitignore
git add .
git push -u origin master
```

### git submodule 相关命令
[git子模块的修改和更新操作_子模块怎么修改上传-CSDN博客](https://blog.csdn.net/xqj2333/article/details/107826608)  
[Git submodule add: "a git directory is found locally" issue - Stack Overflow](https://stackoverflow.com/questions/20929336/git-submodule-add-a-git-directory-is-found-locally-issue)  

在父项目中修改并提交子项目代码  
子项目中有三种状态，一是以版本编号命名的游离态，二是主分支master，三是自己创建的分支。一般是在自己的分支进行修改。先cd到子项目的目录下，然后执行以下语句：
```
git checkout <分支名>
git add .
git commit -m "说明信息"
git push origin <远程分支名>
```

在父项目中更新子项目修改
```
git submodule update --init  // 初始化版本
git submodule update --remote  // 更新到最新版本
```

在父项目中提交子项目版本
```
git submodule update --init
git submodule update --remote
git add 子项目所在文件夹
git commit -m "说明信息"
git push
```

### 更新主题
[github-style theme](https://themes.gohugo.io/themes/github-style/)  
```
cd themes/github-style
git pull
```

### 编辑readme

```
hugo new readme.md
echo '`Hello World!`' > content/readme.md
```

## 2.新增帖子

Hugo will create a post with `draft: true`, change it to false in order for it to show in the website.  
```shell
hugo new post/title_of_the_post.md
```

## 3.本地预览
`hugo server -t github-style -D -w`
> hugo server 的选项:  
-w 修改后本地服务器可以立即变化  
-D 也显示 `draft: true`的帖子

### 错误1
如果出现了下面的报错，检查下：  
1.使用的配置文件是哪个  
2.`config.toml`文件中的`themeDir`的路径以及`theme`的名称是否和下载的文件夹名称一致
```
WARN 2019/05/31 16:14:35 found no layout file for "HTML" for "section": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
WARN 2019/05/31 16:14:35 found no layout file for "HTML" for "section": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
WARN 2019/05/31 16:14:35 found no layout file for "HTML" for "taxonomyTerm": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
```
### 错误2
有些主题（比如puppet）依赖hugo名字中带extended的版本：`hugo_extended_0.121.2_linux-amd64.tar.gz`，否则会执行报错。

## 4.部署到 Github Pages

注意确认下 `${username}.github.io` 这个repo下面 `Settings/Pages/Branch` 是否和部署的分支一样。

### 手动部署

生成 `hugo --theme=your_theme --baseUrl="your_server_or_domain" --buildDrafts`  
提交 `public` 目录到 `${username}.github.io` 仓库  
在仓库的 `https://github.com/${username}/${username}.github.io/settings/pages` 页面可以设置部署哪个分支

### 使用 Github Actions 自动部署
[Hugo + GitHub Action，搭建你的博客自动发布系统 · Pseudoyu](https://www.pseudoyu.com/zh/2022/05/29/deploy_your_blog_using_hugo_and_github_action/)

- 在 github 账户申请具备 repo 和 workflow 权限的密钥
- 在博客原始代码仓库添加环境变量 `PERSONAL_TOKEN`  
![image](https://cdn.jsdelivr.net/gh/devin0x01/myimages@master/githubpages/image_a98bd2b10c3990c971f643943b261a8d.png)
- 编辑 [.github/workflow/deploy.yml](https://github.com/devin0x01/myblogs/blob/master/.github/workflows/deploy.yml) 文件

  > `on` 表示 GitHub Action 触发条件，我设置了 `push`、`workflow_dispatch` 和 `schedule` 三个条件:  
  `push`，当这个项目仓库发生推送动作后，执行 GitHub Action  
  `workflow_dispatch`，可以在 GitHub 项目仓库的 Action 工具栏进行手动调用  
  `schedule`，定时执行 GitHub Action，主要是使用一些自动化统计 CI 来自动更新我博客的关于页面，如本周编码时间  

  > `jobs` 表示 GitHub Action 中的任务，我们设置了一个 build 任务，`runs-on` 表示 GitHub Action 运行环境，我们选择了 ubuntu-latest。  
  我们的 build 任务包含了 Checkout、Setup Hugo、Build Web 和 Deploy Web 四个主要步骤。`run` 是执行的命令，`uses` 是 GitHub Action 中的一个插件，我们使用了 `peaceiris/actions-hugo@v2` 和 `peaceiris/actions-gh-pages@v4` 这两个插件。Checkout 步骤中 `with` 配置 `submodules` 值为 `true` 可以同步博客源仓库的子模块，即我们的主题模块。  
  另外，`EXTERNAL_REPOSITORY` 需要改为自己的 GitHub Pages 仓库。

## 5.图床设置
[图床设置教程](http://www.duheweb.com/post/20210421125522.html)  
- picgo-plugin-github-plus: picgo自带的github图床删除图片时不能同步到github，使用此插件可以在picgo相册中删除图片时自动把github图床内的图片也删除了。
- picgo-plugin-rename-file: 此插件可以对上传的文件按指定格式重命名，比如按照md5值等。

### 图床访问加速
[CDN jsdelivr加速github图床](https://finisky.github.io/speedupgithubbycdn)  
替换前缀即可, 其中`@{branch}`部分可以省略：replace `https://raw.githubusercontent.com/{user}/{repo}/{branch}/`
to `https://cdn.jsdelivr.net/gh/{user}/{repo}@{branch}/`
