## docker 入门简介

### 云计算平台

- IaaS
	- 虚拟机 存储 负载均衡 网络
- PaaS
	- 运行时环境 数据库 web服务器 开发工具
- SaaS
	- 客户关系管理 邮件 虚拟桌面 通信 游戏

#### IaaS
理解为基础设施运维人员服务，提供计算 存储 网络以及其他基础资源，云平台使用者可以在上面
部署和运行包括操作系统和应用程序在内的任意软件，无需再为基础设施的管理而分心

#### Paas
应用开发人员服务，提供支撑应用运行所需要的软件运行时环境，相关工具与服务，如数据库服务，
日志服务 监控服务等，让应用开发者可以专注于核心业务的开发

#### SaaS
一般用户服务，提供了一套完整可用的软件系统，让一般用户无需关注技术细节，只需要通过浏览器
应用客户端等方式 就能使用部署在云上的应用服务

### 容器
最新的容器技术引入了 OpenVZ Solaris Zones 以及 Linux容器（如lxc）使用这些新技术，
容器不再仅仅是一个单纯的运行环境，在自己的权限范围内，容器更像是一个完整的宿主机，对Docker
来说，它得益于现在Linux内核特性，如控制组(control group),命名空间(namespace) 技术，
容器和宿主机之间的隔离更加彻底，容器有独立的网络和存储栈，还拥有自己的资源管理能力，使得
同一台宿主机中的多个容器可以友好的地共存

容器需要的开销资源有限，和传统的虚拟化以及半虚拟化技术(paravirtualization)相比，容器
运行不需要模拟(emulation layer）和管理层（hypervisor layer），而是使用操作通的系统
调用接口，这降低了运行单个容器所需要的开销，也使得宿主机中可以运行更多的容器

### 容器云

容器云以容器为资源分割和调度的基本单位，封装整个软件运行时环境 为开发者和系统管理员提供
用于构建 发布和运行分布式应用的平台。当容器云专注于资源共享和隔离，容器编排与部署时候
它更近传统的Iaas。当容器云渗透到应用支撑与运行是的环境时， 它更接近于传统的PaaS.

从容器到容器云是一种伟大的进化，并依旧在日积月累中不断前行，现在让我们一起进入Docker世界
感受容器和容器云的魅力

### docker 简介
Docker是一个能够把开发的应用程序自动部署到容器的开源引擎。用于构建 发布 和运行分部署
应用的平台，它是一个跨平台 可移植并且简单易用的容器解决方案。

Docker代码托管在GitHub上，基于Go语言开发 并遵从Apache 2.0协议，通过操作系统内核技术(namespaces cgroups)等 为容器提供资源隔离与安全保障。

Docker项目是由Solomon Hykes 所带领的团队发起，在Docker公司的前身dotCloud内部启动孕育
代码托管在GitHub。

2013年3月：Docker正式发布开源版本。

### docker 特点

- 持续部署与测试
- 跨平台支持
- 环境标准和版本控制
- 高资源利用率与隔离
- 容器跨平台性与镜像
- 易于理解且易用
- 应用镜像仓库

### Containers and virtual machines
![image](http://upload-images.jianshu.io/upload_images/1542757-4a4d2dfb279f95eb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### docker 客户端
Docker是一个典型的C/S架构的应用程序，但在发布上 Docker将客户端和服务器端统一在同一个
二进制文件中，不过 这只是对于Linux系统而言的 在其他平台上如Mac上，Docker只提供了用户
端

Docker客户端一般通过Docker Command来发起请求，另外 也可以通过Docker提供的一整套
Restful API来发起请求， 这种方式更多地被应用在应用程序的代码中
### docker daemon
Docker daemon也可以被理解成DockerServer，另外 人们也常常用Docker Engine来直接
描述它，因为这实际上就是驱动整个Docker功能的核心引擎

简单的说，Docker daemon实现的功能就是接收客户端来的请求，并实现请求所要求的功能，
同时针对返回相应的结果，在功能的实现上，因为涉及了容器 镜像 存储等多方面的内容
daemon内部的机制会复杂很多，涉及多个模块之间的实现和交互
### docker 镜像
可以理解为类似于传统虚拟化的iso镜像，不过Docker镜像相对要轻量化很多，它只是一个可以
定制的rootfs。Docker镜像的另一个创新是 它是层级的 并且是可复用的。如果是基于相同的
发行版的镜像，在大多数文件的内容上都是一样的，基于此，当然会希望可以服用他们。
利用Unionfs的特性，Docker会极大迪减少磁盘和内存的开销

docker 镜像可以通过Dockerfile来创建的，Dockerfile提供了镜像内容的定制，同时也
体现了曾经关系的建立。也可以通过使用docker commit命令来手动将修改后的内容生成镜像
这些将在后面详细介绍

### Registry
Registry是一个存放镜像的仓库，它通常部署在互联网服务器上或者云端上

Docker公司提供了官方的Registry叫Docker Hub这上面提供了大多数常用软件和发行版的镜像

Registry本身也是一个开源项目，任务人都可以下载进项部署，所以多数企业选择在自己的内部
部署一套自己的Docker Hub 后二次开发。

### docker 生态
![image](http://upload-images.jianshu.io/upload_images/1542757-9aacc500da168482.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
