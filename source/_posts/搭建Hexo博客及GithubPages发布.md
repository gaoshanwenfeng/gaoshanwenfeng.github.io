---
title: 搭建Hexo博客及GithubPages发布
date: 2023-03-12 23:36:52
tags:
  - Hexo
  - GithubPages
categories:  
  - 工具 
comments: true
description: 通过Hexo搭建博客,发布到Github上托管。  
---

## Hexo 的介绍说明
Hexo 是一个快速、简洁且高效的博客框架。 Hexo 使用 Markdown（或其他标记语言）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。
参考官网：[Hexo官方文档](https://hexo.io/zh-cn/docs/)

## 本地环境安装

在开始安装Hexo之前, 需要安装Git和Node.js, Hexo官方文档要求Node版本>=10.13, 最好使用12.0以上的版本, 直接选择最新的版本。

### 安装Git

### 安装NodeJS和NPM
通过命名  "node -v" 和 "npm -v"确认安装的结果和版本

### 安装hexo

所有必备的应用程序安装完成后，即可使用 npm 安装 Hexo。

$ npm install -g hexo-cli
进阶安装和使用
对于熟悉 npm 的进阶用户，可以仅局部安装 hexo 包。

$ npm install hexo
安装以后，可以使用以下两种方式执行 Hexo：

npx hexo <command>
Linux 用户可以将 Hexo 所在的目录下的 node_modules 添加到环境变量之中即可直接使用 hexo <command>：
echo 'PATH="$PATH:./node_modules/.bin"' >> ~/.profile
安装 Hexo 只需几分钟时间，若您在安装过程中遇到问题或无法找到解决方式，请 提交问题，我们会尽力解决您的问题。


初始化完成的hexo_blog文件结构大概是这样的
`tree -L 1`
{% codeblock lang:bash %}
├── _config.yml
├── db.json
├── node_modules
├── package.json
├── package-lock.json
├── public
├── scaffolds
├── source
└── themes
{% endcodeblock %}
其中的_config.yml文件控制着网站的整体配置, source/_posts存放Markdown文章, themes存放主题
## Hexo的配置


### 配置主题them

根据官网的主题列表选择主题，然后根据安装指引安装和配置即可
参考[Hexo主题列表](https://hexo.io/themes/)


选择使用 maupassant 主题
{% codeblock lang:bash %}
git clone git@github.com:tufu9441/maupassant-hexo.git themes/maupassant
npm install hexo-renderer-pug --save
npm install hexo-renderer-sass-next --save
{% endcodeblock %}



## 配置Github

### GitHub Actions 部署 GitHub Pages。 
此方法适用于公开或私人储存库. 若你不希望将源文件夹上传到 GitHub。

1.建立名为 username.github.io的储存库。 若之前已将 Hexo 上传至其他储存库，将该储存库重命名即可。
2.将 Hexo 文件夹中的文件 push 到储存库的默认分支。 默认分支通常名为main，旧一点的储存库可能名为master。
将 main 分支 push 到 GitHub：
 $ git push -u origin main
  默认情况下 public/ 不会被上传(也不该被上传)，确保 .gitignore 文件中包含一行 public/。 整体文件夹结构应该与 示例储存库 大致相似。
3.使用 node --version 指令检查你电脑上的 Node.js 版本。 记下主要版本（例如，v20.y.z）
4.在储存库中前往 Settings > Pages > Source 。 将 source 更改为 GitHub Actions，然后保存。
5.在储存库中建立 .github/workflows/pages.yml，并填入以下内容 (将 22 替换为上个步骤中记下的版本)：

{% codeblock lang:yaml %}
.github/workflows/pages.yml
name: Pages

on:
  push:
    branches:
      - main # default branch

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 22
        uses: actions/setup-node@v4
        with:
          # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
          # Ref: https://github.com/actions/setup-node#supported-version-syntax
          node-version: "22"
      - name: Cache NPM dependencies
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
{% endcodeblock %}        
部署完成后，前往 username.github.io 查看网页。
若你使用了一个带有 CNAME 的自定义域名，你需要在 source/ 文件夹中新增 CNAME 文件。 更多信息

### 项目页面

如果您希望在 GitHub 上有一个项目页面：

1.导航到 GitHub 上的存储库。 转到 Settings 选项卡。 建立名为 <repository 的名字> 的储存库，这样你的博客网址为 <你的 GitHub 用户名>.github.io/<repository 的名字>，repository 的名字可以任意，例如 blog 或 hexo。
2.编辑你的 _config.yml，将 url: 更改为 <你的 GitHub 用户名>.github.io/<repository 的名字>。
在 GitHub 仓库的设置中，导航至 Settings > Pages > Source 。 将 source 更改为 GitHub Actions，然后保存。
Commit 并 push 到默认分支上。
3.部署完成后，前往 username.github.io/repository 查看网页。

### 一键部署

1.安装 hexo-deployer-git。
   `npm install hexo-deployer-git --save`
2.在 _config.yml 中添加以下配置（如果配置已经存在，请将其替换为如下）:
```bash
deploy:
  type: git
  repo: https://github.com/<username>/<project>
  # example, https://github.com/hexojs/hexojs.github.io
  branch: main
```  
3.执行 hexo clean && hexo deploy 。
4.浏览 username.github.io，检查你的网站能否运作。

## 发布

常用命名
hexo n "我的博客" == hexo new "我的博客"
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy

1、创建页面
{% codeblock lang:bash %}
hexo 
{% endcodeblock %}

2、每次发布前先清理缓存, 使用简化的hexo cl命令
{% codeblock lang:bash %}
hexo cl # 清除缓存
INFO  Validating config
INFO  Deleted database.
INFO  Deleted public folder.
{% endcodeblock %}

3、重新生成静态文件, 可以注意下文档名是否正常转化, 图片是否正常被归档到对应目录
{% codeblock lang:bash %}
hexo g # 生成静态文件
INFO  Validating config
INFO  Start processing
INFO  Path converted: /(?<=[(<\s])(.\/)?Jenkins工作空间和Maven构建缓存清理\//g → /2024/7/3/Jenkins-Workspace-and-Maven-Build-Cache-Cleanup/
INFO  Files loaded in 956 ms
INFO  Generated: archives/index.html
INFO  Generated: index.html
INFO  Generated: img/alipay.svg
...
INFO  42 files generated in 267 ms
{% endcodeblock %}

4、在本机启动hexo server预览效果
{% codeblock lang:bash %}
hexo s # 本地预览
INFO  Validating config
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/ . Press Ctrl+C to stop.
{% endcodeblock %}
预览生成的博客, 确定无误后使用Ctrl+C退出Server

5、配置完成了Github, 最后一步就是上传到你的Repo
{% codeblock lang:bash %}
hexo d #发布
{% endcodeblock %}

大功告成, 访问username.github.io来查看你的博客吧

### 通过Hexo发布


## 编写文档

### 创建文档
创建一篇新文章或者新的页面。
`hexo new [layout] <title>`
post是默认的布局，但你也可以提供自己的布局。 您可以通过编辑 _config.yml 中的 default_layout 设置来更改默认布局。

### 创建标签页面
1、执行命令 `hexo new page tags`
2、Hexo会自动在/source/tags文件夹下生成一个index.md文件。
  `/source/tags/index.md`
3、编辑index.md文件，添加type: tags属性（以及layout: tags，如果适用），以指明这是一个标签页面
{% codeblock %}
---
title: 标签
layout: post
date: 2013-01-01 20:46:25
type: tags
---
{% endcodeblock %}

### 创建分类页
Hexo博客中创建一个专门的“分类”页面。这可以通过Hexo的命令行工具实现：

1.打开命令行工具，进入博客所在的文件夹。 
 `hexo new page categories`
2.Hexo会自动在/source/categories文件夹下生成一个index.md文件。
  `/source/categories/index.md`
3.编辑index.md文件，添加type: categories属性（以及layout: categories，如果适用），以指明这是一个分类页面。
{% codeblock %}
---
title: 分类
layout: post
date: 2013-01-01 20:46:25
type: categories
---
{% endcodeblock %}








