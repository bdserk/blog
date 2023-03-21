# hugo
**Hugo** 是一个用 Go 语言编写的静态网站生成器 类似的静态网站生成器还有Jekyll、hexo等等。以上生成器都使用过，但感觉要么环境麻烦，要么生成静态页面步骤繁琐以及生成缓慢。

如果你正想在GitHub上搭建个静态的博客，搜索一大堆都是关于Jekyll和hexo的相关文章，使用Hugo的相关文章却很少，但是我认为使用Hugo方便一点。本着学习分享的原则，下面整理下如何使用Hugo。
#### 依赖软件版本要求
[go](https://golang.org/dl/)(at least go 1.11)
[git](https://git-scm.com/)(1.8++)

#### 支持平台
Hugo currently provides pre-built binaries for the following:

- macOS (Darwin) for x64, i386, and ARM architectures
- Windows
- Linux
- OpenBSD
- FreeBSD

#### install git 
```
yum install git -y 
```
#### install go 

```
tar xf 
mv go /usr/local/bin/go 
echo "export PATH=$PATH:/usr/local/bin/go"  > /etc/profile.d/go
source /etc/profile.d/go 
```
#### install hugo

[hugo release](https://github.com/gohugoio/hugo/releases)二进制包 下载下来直接解压 
```
wget 
tar xf -C /usr/local/bin/hugo   
echo "export PATH=$PATH:/usr/local/bin/hugo"  > /etc/profile.d/hugo
source /etc/profile.d/hugo 
```


#### 新建站点目录
安装并配置Hugo环境后，打开cmd命令行，可以直接使用hugo命令了
```
[root@bds-aliyun]# hugo verison 
[root@bds-aliyun]#cd /data/application 
[root@bds-aliyun /data/application]# hugo new site hugoblog  #创建站点 
[root@bds-aliyun /data/application]# cd hugoblog 
[root@bds-aliyun hugoblog]# ll
总用量 44
drwxr-xr-x   2 root root  4096 10月 21 11:22 archetypes
-rw-r--r--   1 root root    99 10月 21 11:25 config.toml
drwxr-xr-x   3 root root  4096 10月 21 11:26 content
drwxr-xr-x   2 root root  4096 10月 21 11:22 data
drwxr-xr-x   2 root root  4096 10月 21 11:22 layouts
drwxr-xr-x   2 root root  4096 10月 21 11:26 resources
drwxr-xr-x   2 root root  4096 10月 21 11:22 static
drwxr-xr-x 280 root root 12288 10月 21 14:25 themes
```
创建的站点文件目录说明
```
|- archetypes ：存放default.md，头文件格式

|- content ：content目录存放博客文章（.post/.md文件）

|- data ：存放自定义模版，导入的toml文件（或json，yaml）

|- layouts ：layouts目录存放的是网站的模板文件

|- static ：static目录存放js/css/img等静态资源

|- config.toml ：config.toml是网站的配置文件

当前网站是没有任何内容的，需要下载个主题跑起来才有内容。
```
#### 生成提交一篇文章
文章格式markdown
```
cd /data/application/hugoblog
hugo new posts/my-first-post.md
```
##### Install all Themes
```
git clone --depth 1 --recursive https://github.com/gohugoio/hugoThemes.git themes
```
##### Install sample Themes
```
cd themes
git clone https://github.com/spf13/hyde
```
主题文件夹如下
```
|- archetypes ：存放default.md，头文件格式

|- layouts ：主题模板文件

|- static ：静态资源

|- theme.toml ：主题配置文件

```
修改网站的配置文件 添加所选择的主题
```
echo "theme = "hyde" >> config.toml
```
#### 启动hugo
Hugo内置`http server`，在你的站点根目录执行`hugo server`命令

生成静态文件 
```
$ cd /data/application/hugoblog && hugo
```
查看hugoblog目录会生成public目录
```
[root@bds-aliyun hugoblog]# ll
总用量 44
drwxr-xr-x   2 root root  4096 10月 21 11:22 archetypes
-rw-r--r--   1 root root    99 10月 21 11:25 config.toml
drwxr-xr-x   3 root root  4096 10月 21 11:26 content
drwxr-xr-x   2 root root  4096 10月 21 11:22 data
drwxr-xr-x   2 root root  4096 10月 21 11:22 layouts
drwxr-xr-x   7 root root  4096 10月 21 14:12 public
drwxr-xr-x   2 root root  4096 10月 21 11:26 resources
drwxr-xr-x   2 root root  4096 10月 21 11:22 static
drwxr-xr-x 280 root root 12288 10月 21 14:25 themes
```
启动
```
hugo server --baseURL=http://hugo.budongshu.cn -p 81 --bind=0.0.0.0 
```
访问 http://hugo.budongshu.cn:81/ 
