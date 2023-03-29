

**前言**： 在我们编写个人博客的时候，针对图片的保存一直是一个问题，因为图片是保存在本地，一旦博客进行分享，那么图片就会丢失，所以这里给大家介绍一个靠谱，稳定，又方便的方式实现个人图床，就是Picgo + Gitee 还有结合Typora进行写markdown博客，Typora支持PicGo上传图片，学会使用这几个工具，写博客会方便，简单很多。

### 软件需知

> 首先电脑必须安装nodejs环境(node ,npm)

#### 工具介绍

- `Typora`: Markdown工具，写Markdown文件的神器，简洁、方便、免费
- `PicGo` 开源的图片管理工具，可以自己上传图片到各种图床
- `gitee-uploader`: PicGo依赖这个插件进行上传到`gitee` 仓库
- `gitee码云`: 借助`gitee` 码云建立自己的仓库，构建**免费**图床，国内速度快

#### 软件版本

- `Typora`:  Typora Beta 0.9.9.35.2

- `Picgo`:  v2.3.0-beta.3 + 

- `gitee-uploader`:  1.1.2 

- `gitee`: 申请 gitee 码云平台账号 

  

###  安装nodejs 

`nodejs` 下载地址：https://nodejs.org/en/download/current/ 



![image-20201126000352762](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126000352762.png)



### Picgo 介绍

详情请看github地址： https://github.com/Molunerfinn/

#### Picgo下载 

- 稳定版本

https://github.com/Molunerfinn/PicGo/releases/tag/v2.2.2   

- 测试体验版本，可能存在bug 

https://github.com/Molunerfinn/PicGo/releases/download/v2.3.0-beta.3/PicGo-2.3.0-beta.3.dmg

- 百度云地址（上面俩个版本我都放到了百度云，提供下载）

链接: https://pan.baidu.com/s/1vclMpCSJRvdapdHSO2wkqQ 提取码: tw88 



#### Picgo 安装

> 我这里安装版本是最新Picgo-2.3.0

如果遇到下面报错，请根据提示，进行安装nodejs  

`nodejs` 下载地址：https://nodejs.org/en/download/current/ 

![image-20201125235825678](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201125235825678.png)

#### Picgo 安装成功后，然后右键打开详细窗口 

>  选择gitee

![](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126003056438.png)

**然后右键点击软件(mac),打开详细窗口,然后选择插件设置 安装gitee-uploader**

![image-20201126011800964](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126011800964.png)

**这里需要填写上传到gitee仓库的一些认证条件，下面会进行讲解**

![image-20201126003043875](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126003043875.png)



### gitee 注册申请

`gitee` 地址： https://gitee.com/login 

#### gitee注册登录

![image-20201126001439621](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126001439621.png)



#### gitee 建立自己的图片仓库

<img src="https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126001652453.png" alt="image-20201126001652453"  />



#### gitee 设置仓库信息

>  最后选择进行创建

![image-20201126002327936](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126002327936.png)



#### gitee的私人令牌token 生成

![image-20201126003911644](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126003911644.png)

![image-20201126002749069](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126002749069.png)

这是我的token令牌，进行复制后面会PicGo会用到

![image-20201126002859007](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126002859007.png)

![image-20201126011329717](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126011329717.png)

- `repo`:  比如我的仓库地址: https://gitee.com/budongshu/blogimage     去掉https://gitee.com/   

- `token`: 就是上面获取的私人令牌token  

- `path`：建立一个文件夹来保存图片，这里设置好后，仓库里面会自动创建这个目录

  

**现在就可以上传图片了**

### Typora 设置支持PicGo 

> 设置Typora工具，当插入图片的时候，触发上传图片操作，然后上传服务选择PicGo.app来支持

![image-20201126101750352](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126101750352.png)

![image-20201126101135224](https://gitee.com/budongshu/blogimg/raw/master/img/image-20201126101135224.png)
