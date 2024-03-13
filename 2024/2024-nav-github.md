# 个人导航免费拥有

## nav导航项目
项目地址： <https://github.com/shenweiyan/WebStack-Hugo>

## 部署准备
- 首先fork 项目到自己仓库（项目名字不能更改）
- 按照步骤，申请token，并且按照文档配置，开启action自动部署
- 更改workflows里面的hugoAction.yml脚本，进行自动发布
- 开启github pages，绑定域名
- 域名解析，正常访问

### 1.1 fork 项目
这里忽略，可以参考项目文档操作

### 1.2 配置token和actions
这里忽略，可以参考项目文档操作


### 1.3 配置huoAction.yml自动发布脚本
> 这里是重点关注的地方，当初这里卡了好久

```
name: Hugo Actions # 名字自取

on:
  push:
    branches:
      - main  # 这里的意思是当 main 分支发生 push 的时候，运行下面的 jobs

jobs:
  deploy: 
    runs-on: ubuntu-20.04 	# 在什么环境运行任务
    steps:
      - uses: actions/checkout@v3   # 引用 actions/checkout 这个 action，与所在的 github 仓库同名
        with:
         submodules: true  # Fetch Hugo themes (true OR recursive) 获取 submodule 主题
         fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod
         
      - name: Setup Hugo	# 步骤名自取
        uses: peaceiris/actions-hugo@v2   # Hugo 官方提供的 action，用于在任务环境中获取 hugo
        with:
          hugo-version: '0.122.0'	# 获取指定版本的 hugo
          extended: false

      - name: Build Site
        run: |
          pwd
          ls
          cp -R exampleSite/* .
          hugo --themesDir ../  --baseURL   https://dao.bdser.cc/
      # 创建 CNAME，这个是原始配置中没有的
      - uses: "finnp/create-file-action@master"
        env:
          FILE_NAME: "./public/CNAME"
          FILE_DATA: "dao.bdser.cc"
          
          
      - name: Deploy Pages
        uses: peaceiris/actions-gh-pages@v3	  # 一个自动发布 github pages 的 action
        with:
          external_repository: bdser/WebStack-Hugo	  # 发布到哪个 repo
          personal_token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}	# 发布到其他 repo 需要在对应的 repo 上粘贴生成的 personal_access_token
          #1. 创建 Personal_Access_Token：
          #   GitHub 个人账号 -> Settings -> Developer settings -> Personal access tokens ->  Tokens (classic) -> New personal access token (classic)
          #   - Expiration: No expiration
          #   - Select scopes: 全选
          #2. 粘贴 Personal_Access_Token：
          #   进入目标 repo -> Settings -> Security -> Secrets and variables -> Actions，选择 New repository secret -> 粘贴前面生成的 personal_access_token
          publish_dir: ./public	
          #注意这里指的是 Pages 要发布哪个文件夹的内容，而不是指发布到目标仓库的什么位置；因为 hugo 默认生成静态网页到 public 文件夹，所以这里发布 public 文件夹里的内容。
          publish_branch: gh-pages	# 发布到哪个 branch
          force_orphan: true
          full_commit_message: ${{ github.event.head_commit.message }}
```

`重点是脚本配置`

![](https://gitee.com/budongshu/blogimg/raw/master/img/企业微信截图_5623598e-5650-4212-96a6-0ec9a4063e9c.png)

### 1.4 配置github pages
> github 可以配置多个pages的，需要一定的配置技巧

首先配置仓库分支，然后在Custom Domain里面配置域名（建议使用域名），直接配置成三级域名，然后开启https，如下图
然后dns域名解析的时候，配置cname记录，访问的时候，直接访问下三级域名即可

cname记录: dao.bdaer.cc  CNAME  bdser.github.io
![](https://gitee.com/budongshu/blogimg/raw/master/img/20240312183856.png)

### 1.5 域名访问
修改配置文件，自动发布更新，可以访问<https://dao.bdser.cc> ，查看效果






