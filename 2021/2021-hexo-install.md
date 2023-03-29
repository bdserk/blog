通过hexo 可以搭建属于的免费博客，不需要敲代码，不需要花钱，并且还很好看，但是需要稍稍花上一点时间，当然只要按照文档，一步一步操作，就可以收获属于自己的博客啦



需要的工具介绍

`nodejs`: 包含俩个命令 `node` `npm`

`hexo`: 快速、简洁且高效的博客框架,支持markdown 插件和扩展性很好

`git`：需要安装git 通过git 命令来操作github 

`github`账号：申请注册一个github账号，github提供了一个github pages功能可以支持站点

软件版本： 

`nodejs`： v12.20.0 建立使用这个版本，最新版本使用hexo的时候，可能会有bug

### 开始搭建并部署博客

#### nodejs 操作

##### windows 用户 安装nodejs

>  windows用户这里需要安装git来操作

Git的官方下载地址：https://git-scm.com/download/win 

下载完成之后点击安装，然后通过打开Git Bash 这个软件来敲下面的命令

**临时更换npm 源 (为了后面快速下载安装hexo，更换为国内的源)**

```
npm config set registry https://registry.npm.taobao.org
npm config get registry
看看终端显示信息，如果看到这行信息，说明配置成功: https://registry.npm.taobao.org/
```

##### mac 用户 打开终端 安装nodejs

**下载安装 然后通过命令检测下安装版本**

下载地址： https://npm.taobao.org/mirrors/node/v12.20.0/

```shell
$ node -v
v12.20.0
(base)
$ npm -v
6.14.8
```

**临时更换npm 源 (为了后面快速下载安装hexo，更换为国内的源)**

```shell
$ npm config set registry https://registry.npm.taobao.org
$ npm config get registry
https://registry.npm.taobao.org/

```

#### hexo 操作

##### windows用户 安装初始化hexo

>  通过npm来安装hexo，下面前俩个命令安装需要一点时间，blog目录就是博客目录，根据自己情况选择目录

```shell
npm install hexo-cli -g  
hexo init blog            
cd blog
npm install       
hexo server
```

然后会看到这样的信息

```
INFO  Validating config
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
```

本地访问http://localhost:4000 就可以看到博客网站了

##### mac 用户 打开终端 安装初始化hexo

>  通过npm来安装hexo(mac用户加上sudo),下面前俩个命令安装需要一点时间，blog目录就是博客目录,根据自己情况选择目录

```shell
sudo npm install hexo-cli -g  #安装需要一点时间
hexo init blog   #hexo 初始化安装 需要一点时间 这个blog以后就是自己的博客目录啦 文章都会放在这里面
cd blog          #进入博客目录
npm install      #npm 安装一些插件
hexo server      #启动hexo服务

INFO  Validating config
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop. 
```

![image-20201126122858805](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126122858805.png)

![image-20201126122742048](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126122742048.png)

本地访问http://localhost:4000 就可以看到博客网站了

**注意： 后面不在区分windows和 mac 用户，只要是敲命令的操作，命令都是通用的，命令不要敲错就OK。**

至此，你本地的博客就已经搭建成功，接下来就是部署到 Github Page 了

#### github 操作

##### github 申请账号注册

github 注册地址： https://github.com/join?ref_cta=Sign+up&ref_loc=header+logged+out&ref_page=%2F&source=header-home

![image-20201126113312838](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126113312838.png)

##### **github 登录 新建仓库**

![image-20201126113604379](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126113604379.png)

**github 只能是托管同名代码下 一个静态站点**

![image-20201126114123119](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126114123119.png)

**这里要注意： 我们要更改下默认仓库分支名字 由 main 改为master** 

![image-20201126133223895](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126133223895.png)

![image-20201126133359800](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126133359800.png)



然后打开仓库创建一个 index.html 文件，并随意先写点内容，比如 <h1> 你能看到我 说明部署成功~</h1>

![image-20201126114759280](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126114759280.png)



![image-20201126115032215](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126115032215.png)

![image-20201126115049350](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126115049350.png)

这个时候打开 [http://你的用户名.github.io](http://xn--6qqv7i14ofosyrb.github.io/) 就可以看到你的站点啦，是不是很简单！index.html 内容只是暂时的预览效果，后面把 Hexo 的文件部署上去就可以在 [http://你的用户名.github.io](http://xn--6qqv7i14ofosyrb.github.io/) 看到你自己的博客啦！ 比如我的就是 [http://budongshu.github.io](http://budongshu.github.io/) 了。

![image-20201126115126800](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126115126800.png

#### 配置 SSH key

首先windows用户打开Git-Bash工具 mac 用户打开终端 操作命令都是一样的



执行以下命令全局配置一下本地账户

```shell
git config --global user.name "用户名"
git config --global user.email "邮箱地址"
```

生成一对密钥 SSH key

```shell
ssh-keygen -t rsa  
```

![image-20201126120840429](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126120840429.png)

首次使用还需要确认并添加主机到本机ssh 信任列表 下面可以看到success 返回成功

```shell
 ssh -T git@github.com   
```



![image-20201126121227120](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126121227120.png)

查看刚刚生成公钥 最后要把这个公钥复制粘贴到github上去

```shell
cat ~/.ssh/id_rsa.pub 
```



![image-20201126121614857](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126121614857.png)

然后登录github 在github 上添加刚刚生成的ssh key 把公钥复制粘贴上去



![image-20201126121337851](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126121337851.png)

创建一个新的 SSH key, 标题随便，key 就填刚才生成那个，确认创建，搞定！！这样在你的 SSH keys 列表里就会看到你刚刚添加的密钥

![image-20201126121412755](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126121412755.png)

#### 部署到github 仓库

这个时候我们把本地hexo 和github pages部署已经完成了，接下来我们要把本地的hexo静态站点部署到github pages中，然后通过github pages 我们就可以在互联网上浏览到博客了。



部署之前需要更改hexo 配置 和安装部署插件

第一: 进入blog 目录，打开仓库_config.yml 配置文件，拉到文件末尾，填上如下配置（也可同时部署到多个仓库，比如也可以支持gitee ）：

```shell
# 注意: 这个文件是有格式的，不同层级按照缩进俩个空格，比如： 下面github 根据上面repo 就是俩个空格
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  branch: main
  repo: 
    github: https://github.com/budongshu/budongshu.github.io.git #仓库名字改成自己的
```



第二: 要安装一个部署插件 [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)

```shell
npm install hexo-deployer-git --save
```

第三: 最后执行以下命令就可以部署上传啦，以下 g 是 generate 缩写，d 是 deploy 缩写：

```shell
hexo g -d
```

![image-20201126123828399](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126123828399.png)

稍等一会，在浏览器访问网址： [https://你的用户名.github.io](https://xn--6qqv7i14ofosyrb.github.io/) 就会看到你的博客 

#### hexo 命令

| 命令        | 功能                   |
| :---------- | :--------------------- |
| hexo clean  | 清除所有生成的页面文件 |
| hexo g      | 生成页面               |
| hexo deploy | 推送部署到远程服务器   |
| hexo n xxx  | 新建一篇名为xxx的文章  |

写一遍博客 发布到网上

```shell
hexo new '第一遍文章'     
```

执行完成后可以在 /source/_posts 下看到一个“`第一遍文章.md`” 的文章文件 .md 就是 Markdown 格式的文件 

具体用法可以在网上找一下，语法还是比较简单的。

![image-20201126135451904](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126135451904.png)

然后按照markdown语法 编辑`第一遍文章.md` ，推荐markdown 工具: `Typora` 进行编辑

![image-20201126135355292](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126135355292.png)

编辑完成保存，然后进行发布，注意： 一定要在blog目录下执行命令

```shell
cd ~/blog 
hexo clean && hexo g -d
```



![image-20201126135245477](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126135245477.png)



更换主题fluid 

```shell
cd ~/blog
git clone https://github.com/fluid-dev/hexo-theme-fluid.git themes/fluid 
```

在_config.yml 配置文件中，替换theme 参数，我们 把原来的注释掉加#，然后新增一行theme:  fluid

```shell
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
#theme: landscape
theme: fluid
```

配置主题fluid 

新建about 关于页面

```shell
hexo new page about
```

随便编写介绍自己的about 页面内容

```shell
---
title: about
date: 2020-11-26 16:47:37
layout: about
---
`email`: bdstravel@126.com
```



