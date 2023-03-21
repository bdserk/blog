### 升级openssl 


#### https 网站打分
https://www.ssllabs.com/ssltest/analyze.html?d=www.jd.com
#### openssl 漏洞查询
https://blog.myssl.com/https-security-best-practices-2/
```
mkdir /Data/apps
cd /opt
wget https://www.openssl.org/source/old/1.0.1/openssl-1.0.1t.tar.gz
tar xf openssl-1.0.1t.tar.gz
cd openssl-1.0.1t
./config shared zlib --prefix=/Data/apps/openssl
make
make install
mv /usr/bin/openssl /usr/bin/openssl.old
mv /usr/include/openssl /usr/include/openssl.old
ln -s /Data/apps/openssl/bin/openssl /usr/bin/openssl
ln -s /Data/apps/openssl/include/openssl /usr/include/openssl 
ln -s /Data/apps/openssl/lib/libssl.so.1.1 /usr/lib64/libssl.so.1.1
ln -s /Data/apps/openssl/lib/libcrypto.so.1.1 /usr/lib64/libcrypto.so.1.1
```

```
openssl version
```
### 重新编译nginx  
#### 查看nginx 是否静态编译openssl 
1 nginx -V
 ![image.png](http://upload-images.jianshu.io/upload_images/1542757-ef21bade0e915000.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
发现没有显示出来build openssl 的版本 则表明是静态编译的
2 ldd /Data/apps/nginx/sbin/nginx
```
[root@l-ng5.ops.prod.idc1 openssl]# ldd /Data/apps/nginx/sbin/nginx
	linux-vdso.so.1 =>  (0x00007ffd70ac8000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x0000003014c00000)
	libdl.so.2 => /lib64/libdl.so.2 (0x0000003014400000)
	libcrypt.so.1 => /lib64/libcrypt.so.1 (0x0000003016400000)
	libluajit-5.1.so.2 => /Data/apps/luajit/lib/libluajit-5.1.so.2 (0x00007f78bd38f000)
	libm.so.6 => /lib64/libm.so.6 (0x0000003015400000)
	libpcre.so.0 => /lib64/libpcre.so.0 (0x00007f78bd162000)
	libz.so.1 => /lib64/libz.so.1 (0x0000003015800000)
	libc.so.6 => /lib64/libc.so.6 (0x0000003014800000)
	/lib64/ld-linux-x86-64.so.2 (0x0000003014000000)
	libfreebl3.so => /lib64/libfreebl3.so (0x0000003018800000)
	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x0000003c8ea00000)
# 如果依赖库不含有Openssl，则表明是静态编译的Openssl的。
```

#### 修改下载解压后nginx源码目录里面代码
进入目录
``` 
cd /opt/src/tengine-2.2.0/auto/lib/openssl 
cat conf
``` 
找到下面这样一段代码
```
CORE_INCS="$CORE_INCS $OPENSSL/.openssl/include"
CORE_DEPS="$CORE_DEPS $OPENSSL/.openssl/include/openssl/ssl.h"
CORE_LIBS="$CORE_LIBS $OPENSSL/.openssl/lib/libssl.a"
CORE_LIBS="$CORE_LIBS $OPENSSL/.openssl/lib/libcrypto.a"
```
更改成
```
CORE_INCS="$CORE_INCS $OPENSSL/include"
CORE_DEPS="$CORE_DEPS $OPENSSL/include/openssl/ssl.h"
CORE_LIBS="$CORE_LIBS $OPENSSL/lib/ssleay32.lib"
CORE_LIBS="$CORE_LIBS $OPENSSL/lib/libeay32.lib"
```
#### nginx 编译参数
```
./configure --prefix=/Data/apps/nginx/ --with-debug --add-module=/opt/src/ngx_cache_purge-2.3 --with-http_stub_status_module --with-http_ssl_module --add-module=/opt/src/ngx_devel_kit-0.3.0rc1 --add-module=/opt/src/lua-nginx-module-0.10.2/ --with-ld-opt=-Wl,-rpath,/Data/apps/luajit/lib --with-openssl=/Data/apps/openssl
```
