---
title: 使用hexo和github page 搭建免费blog系统
date: 2023-01-20 16:24:27
tags:
    - 运维
---

我的blog一直使用wordpress。服务器一直也蹭朋友的服务器。朋友的服务器终于要停服了。 我也需要为我很少更新blog找一个永不停服的服务器了。  
最终选定方案为白嫖github page。 使用hexo创建静态的blog页面，然后通过github page免费发布，通过cloudflare CDN进行加加速。  
### hexo 创建静态blog
```shell
yarn global add hexo-cli  #全局安装hexo
mkdir blog && cd blog #创建一个空目录
hexo init #初始一个空项目
yarn install #更新依赖
hexo new "使用hexo和github 搭建免费blog系统" #创建一个新文章
yarn server # 启动本地服务查看blog页面
```
### github 配置
在github创建一个公开项目。将刚创建项目提交github。  
在github action 创建hexo编译发布脚本。  
创建 deploy， 脚本每次master有提交时进行编译，然后将编译结果提交到blog分支。代码如下：
```yml
# This is a basic workflow to help you get started with Actions

name: Blog deploy

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [ master ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v3

      # Runs a set of commands using the runners shell
      - name: Set Node
        uses: actions/setup-node@v3
        with:
          node-version: 16

      - name: Hexo build
        run: |
          yarn install
          yarn clean
          yarn build

      - name: blog deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          publish_branch: blog
          cname: blog.zhangwenjin.com
```
#### github page配置
如图所示
![img](/images/github_page_setting.png)

### cloudflare 配置
在DNS中开启 Proxied 即可  
如果启用了https，需要在在ssl配置中启用full模式，即github负责申请https证书，cloudflare负责转发