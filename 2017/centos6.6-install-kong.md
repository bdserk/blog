### install pgsql 

#### install pgsql yum repo

```shell
wget  https://download.postgresql.org/pub/repos/yum/9.5/redhat/rhel-6-x86_64/pgdg-centos95-9.5-3.noarch.rpm
rpm -ivh pgdg-centos95-9.5-3.noarch.rpm
ls /etc/yum.repos.d/pgdg-redhat-all.repo

```

#### install pgsql9.5 server

```shell
yum install postgresql95 postgresql95-server
/etc/rc.d/init.d/postgresql-9.5 initdb
/etc/rc.d/init.d/postgresql-9.5 start
```

#### config pgsql 验证和listen 监听地址

```
sed  '/all.*127.0.0.1/ s/ident/trust/' /var/lib/pgsql/9.5/data/pg_hba.conf
sed -i  "/^#listen_addresses/a\listen_addresses='*'" /var/lib/pgsql/9.5/data/postgresql.conf
```

####  pgsql hba认证配置

认证权限配置文件为 `/var/lib/pgsql/9.5/data/pg_hba.conf`  
 常见的四种身份验证为：

- **trust**：凡是连接到服务器的，都是可信任的。只需要提供psql用户名，可以没有对应的操作系统同名用户.
- **password** 和 **md5**：对于外部访问，需要提供 psql 用户名和密码。对于本地连接，提供 psql 用户名密码之外，还需要有操作系统访问权。（用操作系统同名用户验证）password 和 md5 的区别就是外部访问时传输的密码是否用 md5 加密.
- **ident**：对于外部访问，从 ident 服务器获得客户端操作系统用户名，然后把操作系统作为数据库用户名进行登录对于本地连接，实际上使用了peer.
- **peer**：通过客户端操作系统内核来获取当前系统登录的用户名，并作为psql用户名进行登录.

####  create pgsql  db user 

```
sudo -u postgres psql
CREATE USER kong; CREATE DATABASE kong OWNER kong;
ALTER USER kong  WITH PASSWORD '123456';
grant all privileges on database kong to kong;/
```



### install kong 

```
cd /opt
wget https://kong.bintray.com/kong-community-edition-rpm/centos/6/kong-community-edition-1.1.0.el6.noarch.rpm
yum install -y epel-release 
yum install -y kong-community-edition-1.1.0.el6.noarch.rpm
```

#### config kong

访问数据库的信息，我们上面创建库和用户和密码时候就是按照下面配置创建的，所以这里用默认配置。 

```
cp  /etc/kong/kong.conf.default /etc/kong/kong.conf
#####截图部分配置
cat /etc/kong/kong.conf         
#pg_host = 127.0.0.1             # Host of the Postgres server.
#pg_port = 5432                  # Port of the Postgres server.
#pg_timeout = 5000               # Defines the timeout (in ms), for connecting,
                                 # reading and writing
#pg_user = kong                  # Postgres user.
#pg_password = 123456            # Postgres user's password.
#pg_database = kong              # The database name to connect to.
```

#### start kong

```
#导入数据
/usr/local/bin/kong migrations bootstrap -c /etc/kong/kong.conf
#启动
/usr/local/bin/kong start -c /etc/kong/kong.conf
#check
curl -i http://localhost:8001/
```

#### kong port

```
Kong默认监听下面端口：
8000，监听来自客户端的HTTP流量，转发到你的upstream服务上。
8443，监听HTTPS的流量，功能跟8000一样。可以通过配置文件禁止。
8001，Kong的HTTP监听的api管理接口。
8444，Kong的HTTPS监听的API管理接口。
```
#### kong dashboard
```
curl --silent --location https://rpm.nodesource.com/setup_9.x | bash -
yum install nodejs -y
npm install -g kong-dashboard
kong-dashboard start   --kong-url http://localhost:8001  --port 8002
```
![image.png](https://upload-images.jianshu.io/upload_images/1542757-51c6b24e6ddc53e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
