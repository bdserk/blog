tengine-lua
<!--more-->

### shell 脚本安装

```
#!/bin/bash
if [ ! -e /Data/apps ];then
      mkdir /Data/apps -pv
fi
if [ ! -e /opt/src ];then
	mkdir /opt/src
fi
cd /opt/src

#tengine

if [ ! -e /opt/src/tengine-2.2.0.tar.gz ];then
	wget http://tengine.taobao.org/download/tengine-2.2.0.tar.gz
	tar xf tengine-2.2.0.tar.gz
fi
#ngx_purge

if [ ! -e /opt/src/ngx_cache_purge-2.3.tar.gz ];then
	wget http://labs.frickle.com/files/ngx_cache_purge-2.3.tar.gz
	tar xf ngx_cache_purge-2.3.tar.gz
fi
#ngx_devel_kit

if [ ! -e /opt/src/v0.3.0rc1.tar.gz ];then
	wget https://github.com/simpl/ngx_devel_kit/archive/v0.3.0rc1.tar.gz
	tar xf v0.3.0rc1.tar.gz
fi
#luajit
if [ ! -e /opt/src/LuaJIT-2.0.4.tar.gz ];then
	wget http://luajit.org/download/LuaJIT-2.0.4.tar.gz
	tar xf LuaJIT-2.0.4.tar.gz
fi
#lua-nginx-module
if [ !  -e /opt/src/v0.10.2.tar.gz  ];then
	wget https://github.com/openresty/lua-nginx-module/archive/v0.10.2.tar.gz
	tar xf v0.10.2.tar.gz
fi
if [ ! -e /Data/apps/luajit  ];then
	cd LuaJIT-2.0.4
	make
	make install PREFIX=/Data/apps/luajit
fi

if [ !  -e /etc/profile.d/luajit.sh ];then
cat > /etc/profile.d/luajit.sh  <<EOF
export LUAJIT_LIB=/Data/apps/luajit/lib
export LUAJIT_INC=/Data/apps/luajit/include/luajit-2.0/
EOF
fi

test -e /etc/profile.d/luajit.sh   &&  . /etc/profile.d/luajit.sh
test -e /Data/apps/luajit/lib/libluajit-5.1.so.2  && ln -s /Data/apps/luajit/lib/libluajit-5.1.so.2 /lib64/

#nginx
cd /opt/src/tengine-2.2.0
./configure --prefix=/Data/apps/nginx/  --with-debug --add-module=/opt/src/ngx_cache_purge-2.3 --with-http_stub_status_module --with-http_ssl_module --add-module=/opt/src/ngx_devel_kit-0.3.0rc1 --add-module=/opt/src/lua-nginx-module-0.10.2/    --with-ld-opt=-Wl,-rpath,/Data/apps/luajit/lib
make
make install

[  $? -eq 0 ] &&  echo 'The tengine lua update Success'
```

### nginx配置文件
#### server 主机配置
```
location /lua {
	        # MIME type determined by default_type:
	        default_type 'text/plain';
	        content_by_lua "ngx.say('Hello World Lua!')";
}
```
#### web页面访问

![image.png](http://upload-images.jianshu.io/upload_images/1542757-b7b8fcf6850ff1de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### fpm打包
#### 目录结构

```
/tmp/nginxinstall/Data/apps/nginx
```
### 脚本

```
[root@budongshu]# cat /tmp/pro.sh
#!/bin/bash
if [ `id avatar |grep avatar | wc -l` -eq 1 ]; then
	exit 0
else
        useradd avatar
	exit 0
fi
```
```
[root@budongshu]# cat /tmp/post.sh
#!/bin/bash
if [  -e /Data/apps/nginx ] ;then
  rm -fr /Data/apps/nginx
  exit 0
else
   exit 0
fi
```
### 命令安装
```
/usr/local/bin/fpm -s dir -t rpm -n hdf-tengine -v 2.2.0 --iteration 3.el6  -d 'pcre,pcre-devel,openssl-devel' --post-install /tmp/pro.sh --post-uninstall /tmp/post.sh  -f -C /tmp/nginxinstall/  --description 'proxy tengine lua 2.2.0 rpm' -p /opt/
```
