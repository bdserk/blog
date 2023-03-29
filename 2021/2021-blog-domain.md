

## 前沿

在之前已经部署博客环境，用hexo初始化博客项目，并且已经把博客托管到了github上，通过github提供的二级域名进行访问

我们自己如果有域名，还可以绑定自己的域名进行访问博客，域名可以通过阿里云进行购买



## 购买域名

这里我们通过阿里云进行购买吧，因为阿里云收购了万网，万网算是国内最大的域名注册商

购买地址: https://wanwang.aliyun.com/domain/searchresult/#/?keyword=&suffix=com

我这里之前购买过域名 budongshu.cn ,下面准备想使用budongshu.cn来访问这个博客

`注意`: **现在大部门域名都是需要 实名认证和备案的，备案的话一般需要3天到7天左右**



![image-20201203102813448](https://gitee.com/budongshu/blogimg/raw/master/image/image-20201203102813448.png)

## 解析域名

这里面有俩种配置方法，但是原理都是一样的

通过cname 来配置 我们现在配置一个@的cname解析和www的cname解析 这样我再浏览器里面输入budongshu.cn(配置@的cname解析起的作用)和www.budongshu.cn 都会解析到budongshu.github.io 也符合我们的预期效果

### 第一种通过cname方式

配置@的cname解析 ，解析记录值是我现在用的访问域名budongshu.github.io 

![](https://gitee.com/budongshu/blogimg/raw/master/image/image-20201203102904219.png)

配置www的cname解析 ，解析记录值是我现在用的访问域名budongshu.github.io 



![image-20201203103112447](https://gitee.com/budongshu/blogimg/raw/master/image/image-20201203103112447.png)

最后的配置效果

![image-20201203103333548](https://gitee.com/budongshu/blogimg/raw/master/image/image-20201203103333548.png)

### 第二种通过解析出来的ip来配置，A记录方式

先通过ping budongshu.github.io 看一下解析ip是多少，我们这里看到是185.199.111.153 ,那么我们同样可以通过A记录的方式

来配置跳转

![image-20201203103428622](https://gitee.com/budongshu/blogimg/raw/master/image/image-20201203103428622.png)

做一个www配置的演示 

![image-20201203103636825](https://gitee.com/budongshu/blogimg/raw/master/image/image-20201203103636825.png)

## 在hexo项目里面设置CNAME

### 1.1  新建文件CNAME

在项目下，进入你的博客项目目录，在source 文件夹下面创建 CNAME 文件（没有后缀名的），填写上域名

可以通过编辑器进行编辑

![image-20201203104545764](https://gitee.com/budongshu/blogimg/raw/master/image/image-20201203104545764.png)



### 1.2 部署博客项目到github

 然后我们部署hexo，通过项目里面的配置会上传到github，这里不明白可以看看我之前hexo 部署文章

```shell
hexo clean && hexo g -d
```



### 1.3 Github 要把https选项勾上

![image-20201203105005191](https://gitee.com/budongshu/blogimg/raw/master/image/image-20201203105005191.png)



## 用自定义的域名访问

完成上述步骤之后就可以在浏览器输入自己的域名访问了,因为我们配置了@ 和www 所以通过下面俩种方式访问

https://budongshu.cn 和 https://www.budongshu.cn 

![image-20201203105252124](https://gitee.com/budongshu/blogimg/raw/master/image/image-20201203105252124.png)

## 找一款自己喜欢的主题

https://hexo.io/themes/ ,这个里面有很多主题，可以任意挑选一个自己喜欢的主题，点击进去通常都有github地址

上面有安装和使用方法介绍，我这里选了一个名叫“fluid” 的主题，这是使用地址：https://github.com/fluid-dev/hexo-theme-fluid

下面截图是fluid使用介绍的部分内容

![image-20201203104039041](https://gitee.com/budongshu/blogimg/raw/master/image/image-20201203104039041.png)

 主题效果

![image-20201203104225967](https://gitee.com/budongshu/blogimg/raw/master/image/image-20201203104225967.png)

可以看到已经实现自定义主题了，主题这里还有很多可以自己设置的地方呢~
