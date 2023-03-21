### Dockerfile 介绍
它是一个文本文件,镜像文件构建脚本
由一些列用于根据基础镜像构建新的镜像文件的专用指令序列组成

### Dockerfile command 

   - ADD
   - COPY
   - ENV
   - EXPOSE
   - FROM
   - MAINTANIER
   - LABEL
   - STOPSIGNAL
   - USER
   - VOLUME
   - WORKDIR

#### FROM 
必须是第一个非注释行,用于指定所用到的基础镜像
##### 语法格式
- FROM <image>[:<tag>]
- FROM <image>@<digest>  
  
**注意: 尽量不要在一个dockerfile中使用多个FROM指令**
####  MAINTANIER 
用于提供信息的指令，用于让作者提供本人的信息
建议紧跟在FROM以后，不限制出现的位置
##### 语法格式
 MAINTANIER <authors detail>
#### COPY
用于docker主机复制文件指向正在创建的映像文件中
##### 语法格式
- COPY SRC DEST
- COPY "SRC"...   "DEST"
##### 解释
- src: 要复制的源文件或目录,支持使用通配符
- dest: 目录路径,正在创建的镜像文件的文件系统路径,建议使用绝对路径,否则,则相对于WORKDIR而言，所有复制生成的目录文件的uid和gid均为0 
##### 注意
- src 必须是build上下文中的路径,因此不能使用类似"../some"的路径
- src 如果是目录,递归复制会自动行,如果有多个src ,包括src上使用了通配符这种情况，此时,dest必须是目录,而且必须以/结尾
- dest 如果事先不存在,它将被自动创建,包括其父目录
#### ADD
类似于COPY指令,额外还支持复制tar文件,以及URL语法
##### 语法格式
- ADD SRC DEST
- ADD "SRC" ... "DEST"
##### 注意
- 以url格式指定的源文件,下载完成后其目标文件的权限为600 
-  如果src是url,且dest不以/结尾,则src指定的文件将被下载并直接被创建为dest, 如果dest以/结尾,则url指定的文件被下载至dest中,并保留原名
- 如果src是一个tar文件,它将被展开为一个目录,其行为类似于tar -x 命令,但是如果通过url下载到的文件是tag格式的,是不会自动被展开的
#### ENV 
定义环境变量,此变量可被当前dockerfile文件中的其它指令调用
调用格式为$varaible_name或者${variable_name}

##### 语法格式
- ENV <key> <value>    一次定义一个变量
- ENV <key>=<value>  ... 一次定义多个变量 如果value中有空白字符,如要用\字符进行转义
##### 注意
- ENV 定义的环境变量在镜像运行的整个过程中一直存在
- 因此可以使用inspect命令查看,甚至可以再docker run启动此镜像时候,使用--env选项来修改制定的环境值

#### USER
指定运行镜像时候, 或运行Dockerfile文件中的任何RUN,CMD,/ENTRYPORINT指定的程序时的用户名或者UID
##### 语法格式
- USER <uid>|<USERNAME> 
##### 注意
- UID 应该使用/etc/passwd  文件存在的用户的UID,否则,docker run可能会出现错误
####  WORKDIR
用于为Dockerfile中所有的指令指定工作目录
##### 语法格式
- WORKDIR <dirpath> 
##### 注意
- WORKDIR 可出现多次,也可使用相对路径,此时表示相对于前一个WORKDIR指令指定的路径 
- WORKDIR 还可以调用ENV定义的环境变量的值

####  VOLUME
用于目标镜像文件中创建一个挂载点的目录,用于挂载主机上的卷或者其它容器的卷
##### 语法格式
- VOLUME <mountpoint>            
- VOLUME ["<mountpoint>" .... ]
#### RUN 
于指定docker build命令过程中运行的命令 
##### 语法格式
- RUN <COMMAND> 
- RUN ["executeable" ,"<param1>" ,"<param2>" ]
##### 注意
- 每个run命令就额外加一层,所以建议一个RUN指令执行多个命令
#### CMD
类似于RUN指令,用于运行程序,但二者运行的是场景不同
CMD在docker run时运行而非docker build,CMD指令的首
要目的为了在于为启动的容器指定默认要运行的程序程序
运行结束, 容器也就结束,不过,CMD指令指定的程序可被
docker run命令行参数中执行的要运行的程序所覆盖
##### 语法格式
- CMD <COMMAND>
- CMD ["executeable" ,"<param1>" ,"<param2>" ]
- CMD ["<para1>","<param2>"] 
##### 注意
- 第三种为ENTRYPOINT指令指定的程序的默认参数   
- 如果dockefile中存在多个CMD指令,只会最后一个生效
#### ENTRYPOINT
类似于CMD,为容器指定默认的启动程序,不会被docker run
所运行的程序所覆盖.而且这些命令行参数,会被当作参数送
给ENTRYPOINT指令指定的程序但是如果运行docker run时,
使用了 --entrypoint选项,此选项的参数可当作要运行的程序覆盖
ENTRYPOINT指令指定的程序
##### 语法格式
- ENTRYPOINT <COMMAND>                           
- ENTRYPOINT ["executeable" ,"<param1>" ,"<param2>" ]
#### EXPOSE 
用于为容器指定要暴露的端口
##### 语法格式
- EXPOSE <PORT>[/<PROTOCAL>] [<PROT></PROTOCAL>]
- PROTOCAL: 默认为tcp,可以是tcp或者udp
##### 例如 eg: 
 - EXPOSE 11211/tcp  11211/udp 

#### ONBUILD
> 定义触发器 

当前dockerfile构建出的镜像被用作基础镜像,去构建其他镜像的时候,ONBUILD指令指定的操作才会被执行
##### 语法格式
- ONBUILD <instruction> 
- ONBUILD ADD my.conf /etc/mysql/my.conf      
##### 注意
ONBUILD不能自我嵌套,且不会触发FROM和MAINTERNATER指令


### docker command 
```
docker build -t bdshello  .     #创建image镜像构建Dockerfile 
docker run -p 4000:80 bdshello  #启动bdshello容器 映射端口
docker run -d -p 4000:80 bdshello  #后台启动
docker container ls             #显示所有启动的容器
docker container ls -a          #显示所有容器包括不启动的
docker container stop <hash id>     #优雅停止容器 
docker container kill  <hash id>    #强制停止容器
docker container rm <hash id>       #移除容器 
docker container rm  $(docker container ls -a -q)  #移除所有容器 
docker image ls -a                #列出所有镜像 
docker image rm <image id>        #删除镜像
docker image rm $(docker image ls -a -q)  #删除所有镜像
```
