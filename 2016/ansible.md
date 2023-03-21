ansible-playbook
### playbook简介
playbook是ansible用于配置，部署，和管理被控节点的剧本。通过playbook的详细描述，执行其中的一系列tasks，可以让远端主机达到预期的状态。playbook就像Ansible控制器给被控节点列出的的一系列to-do-list，而被控节点必须要完成。
也可以这么理解，playbook 字面意思，即剧本，现实中由演员按照剧本表演，在Ansible中，这次由计算机进行表演，由计算机安装，部署应用，提供对外服务，以及组织计算机处理各种各样的事情。

### Playbook使用场景
执行一些简单的任务，使用ad-hoc(调用各种模块,通过命令行来执行命令)命令可以方便的解决问题，但是有时一个设施过于复杂，需要大量的操作时候，执行的ad-hoc命令是不适合的，这时最好使用playbook，就像执行shell命令与写shell脚本一样，也可以理解为批处理任务，不过playbook有自己的语法格式，一会会介绍。
使用playbook你可以方便的重用这些代码，可以移植到不同的机器上面，像函数一样，最大化的利用代码。在你使用Ansible的过程中，你也会发现，你所处理的大部分操作都是编写playbook。


### playbook格式
playbook由YMAL语言编写。YAML参考了其他多种语言，包括：XML、C语言、Python、Perl以及电子邮件格式RFC2822，Clark Evans在2001年5月在首次发表了这种语言，另外Ingy döt Net与Oren Ben-Kiki也是这语言的共同设计者。

YMAL格式是类似于JSON的文件格式，便于人理解和阅读，同时便于书写。首先学习了解一下YMAL的格式，对我们后面书写playbook很有帮助。以下为playbook常用到的YMAL格式。

YMAL语法请参考http://docs.ansible.com/YAMLSyntax.html


### yaml格式语法简介
对于 Ansible, 每一个 YAML 文件都是从一个列表开始. 列表中的每一项都是一个键值对, 通常它们被称为一个 “哈希” 或 “字典”. 所以, 我们需要知道如何在 YAML 中编写列表和字典.

- 文件第一行以 --- (三个连字符)开始,表明yaml文件的开始
- 在同一行中, #之后的内容表示注释, 类似于shell,python
- yaml中的列表元素以 - 开头然后紧跟着一个空格,后面为元素的内容
例子如下:
```
---
- apple
- red
- green
```
- 同一个列表中的元素应该保持相同的缩进。否则会被当做错误处理。
- play中hosts，variables，roles，tasks等对象的表示方法都是键值中间以":"分隔表示,":"后面还要增加一个空格。
- YMAL的有很多的字符串可以解释为true或false：
```
YMAL Truethy: true ,True ,TRUE ,yes ,Yes , YES ,on ,On ,ON ,y ,
YMAL falthy:false ,False ,FALSE ,no ,No ,NO ,off ,Off ,OFF , n ,N
```
- 尽管 YAML 通常是友好的, 但是下面将会导致一个 YAML 语法错误:
```
foo: somebody said I should put a colon here: so I did
```
- 你需要使用引号来包裹任何包含冒号的哈希值, 像这样:
```
foo: "somebody said I should put a colon here: so I did"
```
然后这个冒号将会被结尾.
-  Ansible 使用 “{{ var }}” 来引用变量. 如果一个值以 “{” 开头, YAML 将认为它是一个字典, 所以我们必须引用它, 像这样:
foo: "{{ variable }}
```
---
# 一位职工的记录
name: Example Developer
job: Developer
skill: Elite
```
**让我们把目前所学到的 YAML 例子组合在一起.
 这些在 Ansible 中什么也干不了 但这些格式将会给你感觉:**
```
---
# 一位职工记录
name: Example Developer
job: Developer
skill: Elite
employed: True
foods:
- Apple
- Orange
- Strawberry
- Mango
languages:
ruby: Elite
python: Elite
dotnet: Lame
```
> 一个家庭记录

![clip_image002.png](http://upload-images.jianshu.io/upload_images/1542757-361690a77a4535d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

#### playbook基础

现在的/etc/ansible/hosts配置
```
[root@note1 ansible]# cat /etc/ansible/hosts
[web]
192.168.70.51
[db]
192.168.70.50
```
> 安装一个mysql服务的案列

![image.png](http://upload-images.jianshu.io/upload_images/1542757-9f6e29040930a88a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)
> 在mysql.yml中，主要由三个部分组成:

- hosts部分：使用hosts指示使用哪个主机或主机组来运行下面的tasks，每个playbook都必须指定hosts，hosts也可以使用通配符格式。主机或主机组在inventory清单中指定，可以使用系统默认的/etc/ansible/hosts，也可以自己编辑，在运行的时候加上-i选项，指定清单的位置即可。在运行清单文件的时候，–list-hosts选项会显示那些主机将会参与执行task的过程中。
- remote_user：指定远端主机中的哪个用户来登录远端系统，在远端系统执行task的用户，可以任意指定，也可以使用sudo，但是用户必须要有执行相应task的权限。
- tasks：指定远端主机将要执行的一系列动作。tasks的核心为ansible的模块，前面已经提到模块的用法。tasks包含name和要执行的模块，name是可选的，只是为了便于用户阅读，不过还是建议加上去，模块是必须的，同时也要给予模块相应的参数。

#### playbook运行结果解析

使用ansible-playbook运行playbook文件，得到如下输出信息，输出内容为JSON格式。并且由不同颜色组成，便于识别。一般而言

- 绿色代表执行成功，系统保持原样

- 黄色代表系统代表系统状态发生改变

- 红色代表执行失败，显示错误输出。

> 执行的时候是用ansible-playbook而不是ansible命令了

![image.png](http://upload-images.jianshu.io/upload_images/1542757-531d555823fdb026.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)
> 查看结果

![image.png](http://upload-images.jianshu.io/upload_images/1542757-479c8b4faeb2b253.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

#### 列出执行的远程主机
```
[root@note1 ansible]# ansible-playbook mysql_ansible.yaml --list-hosts
playbook: mysql_ansible.yaml
play #1 (db): host count=1
192.168.70.50
```
#### ansible具有幂等性

> 再次执行的时候ansible会检测这个task是否已经执行过,
   如果这个task任务执行过,它不会再次执行task任务,
   而是直接显示ok状态.


![image.png](http://upload-images.jianshu.io/upload_images/1542757-4467d87410017e11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

###  ansible-playbook 进阶特性
#### playbook 组成结构

```
Invertory    
Modules
Ad Hoc Commands
PlayBooks
Tasks  任务 即调用模块完成的某操作
Variable  变量
Templates 模板 根据客户端的情况来生成一些的数据
Handlers  处理器 由某条件满足能触发执行的操作
Roles  角色
```
#### playbook 基础组件

##### Hosts和Users

playbook中的每一个play的目的都是为了让某个或者某些主机以某个指定的用户身份来执行任务，hosts是用于指定要执行的指定任务的主机，其可以是一个或者多个以冒号分割的主机组，remote_users则用于指定远程主机上的执行任务的用户，如下面所示
```
- hosts: webnodes
 remote_user: root
```
不过 remote_users 也可以用于每个task中，也可以通过指定其通过sudo的方式在远程的主机上执行任务，其可以于play全局或者某任务中，此外，甚至可以在sudo时使用sudo_user 指定sudo时切换的用户
```
- hosts: webnodes
  remote_user: bds
  tasks:
  - name: test connection
    ping:
    remote_user: bds   
    sudo: yes
```
#####  任务列表和action

- play的主体部分是task list task list中的各任务按次序逐个在hosts中指定的所有主机上执行，即在所有主机上完成第一个任务后在开始第二个任务，在运行自上而下某个playbook时，如果中途发生错误，所有已经执行的任务都会讲回滚，因此，在更正playbook后重新执行一次即可

- task目的是使用指定的参数执行模块，而在模块参数中可以使用变量，模块执行是具有幂等性的，这意味着多次执行是安全的，因为其结构均一致

- 每个task都应该有其name ，用于playbook执行结果输出，建议其内容尽可能清晰地描述任务执行步骤，如果未提供name，则action的结果将用于输出

- 定义task的可以使用“action： module option”或者module: options

> 推荐使用后者以实现向后兼容，如果action 一行的内容过多，也可以使用在行首使用几个空白字符进行换行。

```
tasks:
- name: make sure apache is running
  service: name=httpd state=running
```
> 在众多模块中，只用command和shell模块仅需要给定一个列表而无需使用“key=value”格式例如

```
tasks:
- name: disable selinux
  shell: /sbin/setenforce 0
```
> 如果命令或者脚本的退出码不为零，可以使用如下的方式替代

```
tasks:
- name: run this command and igonre the result
  shell: /usr/bin/somecomand || /bin/true
```
> 或者使用ignore_errors来忽略错误信息

```
tasks:
- name: run this command and ignore the result
  shell: /usr/bin/somecommand
  ingore_errors: True
```
> 写一个示例:

```
[root@bj-idc-15 playbooks]# cat /etc/ansible/hosts    #现在hosts配置,新添加一台主机
[web]
192.168.122.52
[db]
10.10.10.15
10.10.10.14
```
```
[root@bj-idc-15 playbooks]# cat nginx.yml
---
- hosts: web
  remote_user: root
  tasks:
  - name: create nginx group
    group: name=nginx  system=yes gid=208
  - name: create nginx user 
    user: name=nginx system=yes uid=208         group=nginx
- hosts: db
  remote_user: root
  tasks:
  - name: copy file to dbservers
    copy: src=/etc/inittabdest=/tmp/inittale.ansible
```
> 执行结果

![image.png](http://upload-images.jianshu.io/upload_images/1542757-ca7939bb86b92d88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

#####  handlers
用于当关注的资源发生变化时采取一定的操作,notify 这个action可用于在每个play的最后被触发，这样可以避免多次有改变发生的时候每次都执行指定的操作,取而代之,仅在所有的变化发生完成时最后一次性的执行指定的操作,在notify中列出的操作成为handlers,也即为notify中调用handler中定义的操作
 ```
- name: template configuration file
   template: src=template.j2 dest=/etc/foo.conf
   notify:
     - restart memcached
     - restart apache
```
> handler是task列表，这些task与前述的task并没有本质上的区别

```
handlers:
- name: restart memcached      #名字要和上面notify的 一致
  service: name=memcached state=restarted
- name: restart apache
  service: name=apache state=restarted
```
> 简单案例: 没有handler的时候
```
[root@bj-idc-14 playbooks]# cat apache.yml
---
- hosts: web
  remote_user: root
  tasks:
  - name: install httpd package
    yum: name=httpd state=latest
  - name: install configurest file
    copy: src=conf/httpd.conf dest=/etc/httpd/conf/httpd.conf
  - name: start httpd service
    service: name=httpd enabled=true state=started
```
> 简单案例：
 
当有一个需求，需要改变httpd配置文案的时候，那么这个时候是需要重启httpd服务的，这时候就需要handlers了，只有某个条件满足的时候才执行
```
[root@bj-idc-14 playbooks]# cat apache.yml
---
- hosts: web
  remote_user: root
  tasks:
  - name: install httpd package
    yum: name=httpd state=latest
  - name: install configurest file
    copy: src=conf/httpd.conf dest=/etc/httpd/conf/httpd.conf
    notify:
      - restart httpd
  - name: start httpd service
    service: name=httpd enabled=true state=started
handlers:
  - name: restart httpd
    service: name=httpd state=restarted
```
##### Templates模板

如果说俩个webservser 安装httpd node1监听80端口，node2 监听8080端口，node1使用的maxClient=100 node2的maxClient=200，这样就需要俩个配置，非常不方便，这样就可以尝试用变量的方式来解决
> 简单案例: 模板中定义变量
```
[root@bj-idc-14 playbooks]# cat templates/httpd.conf.j2  | grep {{    #httpd.conf.j2就是httpd的配置文件
MaxClients       {{ maxclient }}
MaxClients       {{ maxclient }}
Listen  {{ http_port }}
ServerName {{ ansible_fqdn }}    # facts 变量
```
```
[root@bj-idc-14 playbooks]# cat apache.yml
---
- hosts: web
  remote_user: root
  vars:
  - http_port: 888        #定义的变量值 ,然后查看主机的配置文件是否为这里的值
  - maxclient: 305        #定义的变量值 ,然后查看主机的配置文件是否为这里的值
  tasks:
  - name: install httpd package
    yum: name=httpd state=latest
  - name: install configurest file
    copy: src=conf/httpd.conf dest=/etc/httpd/conf/httpd.conf
  #把带变量的模板替换正在运行的配置文件,然后通知httpd重启加载新的配置文件
    template: src=templates/httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf
    notify:
    - restart httpd
  - name: start httpd service
    service: name=httpd enabled=true state=started
handlers:
  - name: restart httpd
    service: name=httpd state=restarted
```
> 执行结果
``` 
[root@bj-idc-14 playbooks]# cat /etc/httpd/conf/httpd.conf |grep -v "^#"| grep ServerName
ServerName bj-idc-14
[root@bj-idc-14 playbooks]# cat /etc/httpd/conf/httpd.conf |grep -v "^#"| grep Listen
Listen 888
[root@bj-idc-14 playbooks]# cat /etc/httpd/conf/httpd.conf |grep -v "^#"| grep MaxClients
MaxClients       305
MaxClients       305
```
### ansible-playbook 中yaml基础元素

- 变量
- Invertory
-  条件判断
- 迭代机制

####  变量

- 变量命名
> 变量名仅能有字母，数字和下划线组成，且只能以字母开头
 
- facts
> facts是由正在通信的远程目标主机发回的信息，可以直接引用, 这些信息保存在ansible变量中，要获取指定的远程主机所支持的所有facts，可试用如下命令进行ansible hostname -m setup
 
- register
> 把任务的输出定义为变量,然后用于其他任务,示例如下:
 ```
tasks:
shell: /usr/bin/foo
register: foo_result
ignore_error: True
 ```
-  通过命令传递变量
> 在运行playbook的时候,也可以传递一些变量供playbook使用,示例如下
```
ansible-playbook test.yaml –extra-vars  “hosts=wwwuser=bds”
 ```

- 通过roles传递变量
> 当给一个主应用角色的时候可以传递变量,然后在角色内使用这变量,如下
```
- host: webservers
  roles:
    - commn
    - { role: foo_arpp_instance , dir: ‘web/htdocs/a.com’ , port: 8080 }
``` 
#### 变量实战
> vars的简单案例
```
[root@bj-idc-14 playbooks]# cat http.yml
---
- hosts: web
  remote_user: root
  vars:
  - groupuser: httpd
  - username: httpd
  tasks:
  - name: create {{ groupuser }} group
    group: name={{ groupuser }} system=yes gid=238
  - name: create {{ username }} user
    user:name={{ username }} system=yes uid=238 group={{ groupuser  }} 
```
> 执行结果

![image.png](http://upload-images.jianshu.io/upload_images/1542757-73246f5d1f6d6c39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

> facts的简单案例: 定义主机变量

```
[root@bj-idc-14 ansible]# cat hosts
[web]
10.10.10.14   httpvars='10.10.10.14'
[db]
10.10.10.15
```
```
[root@bj-idc-14 playbooks]# cat facts_host.yml
---
- hosts: web
  remote_user: root
  tasks:
  - name: copy file 
    copy: content="{{ ansible_all_ipv4_addresses }}, {{ httpvars }} ,
    {{ ansible_cmdline.LANG  }} " dest=/tmp/vars.ansible
```
```
[root@bj-idc-14 playbooks]# ansible-playbook facts_host.yml
```

> 执行结果

![image.png](http://upload-images.jianshu.io/upload_images/1542757-cbb05fef8be5a025.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

####  Inventory

ansible的主要功能用在于批量操作主机，
为了便捷的使用其中的部分主机，
可以在Inventory file中将其分组命名，
默认的Inventory file为/etc/ansible/hosts
Inventory file 可以有多个
也可以通过DynamicInvertory 来动态生成
 
##### Inventory文件格式
Inventory文件遵循INI文件风格，中括号中的字符为组名，可以将同一个主机同时归并到多个不同的组中，此外，当如若目标主机使用了非默认的ssh端口，还可以在主机名称之后使用冒号加端口来表明
例子:
```
ntp.bds.com
[webservers]
www1.bds.com
www2.bds.com
[bdservers]
db1.bds.com
db2.bds.com
```
如果主机名称遵循相识的命名模式，还可以使用列表的形式标识各主机
例子：
```
[webservers]
www[0-51].bds.com
[dbservers]
db-[a-f].bds.com
```
 
##### 主机变量
可以再Inverntory中定义主机时，为其添加主机变量，以便于在playbook中使用例子:
```
[webservers]
www1.bds.com http_port=80 maxRequestsPerChild=1024
```
##### 组变量
组变量是指赋予给指定的组内所有的主机上的在playbook中可用的变量
例子:
```
[webservers]   #可以调用下面[webservsers:vars]中的变量
www1.bds.com
www2.bds.com
[webservsers:vars]
ntp_servser=ntp.bds.com
nfs_servser=nfs.bds.com
```
##### 组嵌套

Inventory中，组还可以包含其他的组，并且也可以向组中的主机指定变量，不过，这些变量只能在ansible-playbook中使用，而ansible不支持
例子:
```
[apache]
http1.bds.com
http2.bds.com
[nginx]
ngx1.bds.com
ngx2.bds.com
[webservsers:children]
apache
nginx
[webservsers:vars]
ntp_server=ntp.bds.com
``` 
##### Inventory参数
ansible 基于ssh连接Invertory中指定的远程主机时，还可以通过参数指定其交互方式，这些参数如下所示
```
基本结构
-host: web
remote_user: root
tasks:
  - task1
    modulesname: module_args
- task2
   -  host: dbserver
```
####  条件测试

如果需要根据变量，facts或者此前任务的执行结果来作为某个tasks执行与否的前提时要用到的条件测试
 
##### when语句
在task后添加when子句即可使用条件测试，
when语句支持Jinjia2表达式语法，例如

```
tasks:
  - name: "shutdown Debian falovred systems"
    command: /sbin/shutdown -h now
    when: ansible_os_family == "Debian"
``` 
when语句中可以使用Jinja2的大多filter功能，
例如要忽略此前某语句的错误并基于其结果
(failed或者sccess)运行后面指定的语句，
可使用类似如下形式
```
tasks:
- command: /bin/false
  register: result
  ignore_errors: True
- command: /bin/something
  when: result| failed
- command: /bin/something_else
  when: result|success
- command: /bin/still/something_else
  when: result| skipped
``` 
此外when语句还可以使用facts或者playbooks中定义的变量
 
> 简单案例

```
[root@bj-idc-14 playbooks]# ansible web -m setup  |grep fqdn
"ansible_fqdn": "bj-idc-14",
[root@bj-idc-14 playbooks]# cat when.yml 
---
- hosts: web
  remote_user: root
  vars:
    - username: user10
  tasks:
- name: create {{ username }}user
  user: name={{ username }} uid=510
  when: ansible_fqdn == "bj-idc-14"
```
> 执行结果

![image.png](http://upload-images.jianshu.io/upload_images/1542757-a10d10e2195cb132.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

#### 迭代机制
当有需要重复性的执行的任务的时候，可以使用迭代机制，
其使用格式为将需要迭代的内容定义为item变量引用，
并通过with_items 语句来指定迭代的元素列表即可
例如：
``` 
- name: add serveral users
  user: name={{ item }} state=present groups=wheel 
  with_items:
    - testuser1
    - testuser2
``` 
上面的语句等同于下面的语句
```
- name: add serveral users1
  user: name=testuser1 state=present groups=wheel 
- name: add serveral users2
  user: name=testuser1 state=present groups=wheel 
```
事实上，with_items中可以使用元素还可以为hashes，例如
```
- name: add serveral users
  user: name={{ item.name }} state=present groups={{ item.groups}} 
  with_items:
    - { name: 'testuser1', groups: 'wheel' } 
    - { name: 'testuser2', gorups: 'wheel' }
```
ansible的循环机制还有很多高级功能,具体参见官方文档
迭代: 表示重复同类task时使用
```
调用: item	
定义循环列表: with_items
		- apache
		- php
		- mysql
```

注意: with_items中的列表值可以是字典，但是引用的时候是使用item.KEY
```
- {name: apache, conf: conffiles/httpd.conf } 
- {name: php , conf: conffiles/php.ini } 
- {name: mysql-servser, conf: conffiles/my.conf }
```
#### ansible-playbook 小技巧

##### Tags

- tags用于让用户选择运行或者路过playbook中的部分代码

- ansible具有幂等性，因此会自身跳过没有变化的部分

- 即便如此,有些代码为测试其确实没有发生变化的时间依然会非常的地长,此时,如果确实其没有发生变化,就可以通过tags跳过这些代码片段当改变配置文件的时候

- 对于安装软件的操作就不需要在执行了,这时候有了tags就可以标记你期望运行的task任务中

- tags表示在playbook可以为某个任务或者某些任务定义一个标签 

- 在执行次playbook时候通过为ansible-playbook 命令使用--tags选项能实现仅仅运行指定的tasks而非所有的

> 简单案例
```
[root@bj-idc-14 playbooks]# cat apache.yml 
--- 
- hosts: web
  remote_user: root
  vars: 
  - http_port: 888 
  - maxclient: 305
  tasks:
  - name: install httpd package
    yum: name=httpd state=latest
  - name: install configurest file 
    #copy: src=conf/httpd.conf dest=/etc/httpd/conf/httpd.conf 
    template: src=templates/httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf
    tags: 
    - config  			 #定义了一个tags 叫做config 
    notify: 
    - restart httpd 
  - name: start httpd service 
    service: name=httpd enabled=true state=started  
    tags: 
    - always               #tags特殊用法 代表总是执行这个任务
handlers: 
  - name: restart httpd 
    service: name=httpd state=restarted   
```
> 执行结果

![image.png](http://upload-images.jianshu.io/upload_images/1542757-b5e8ad8d58c5b0a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

#### check检测和语法检测
不要做任何改变，相反，试着预测一些可能发生的变化

![image.png](http://upload-images.jianshu.io/upload_images/1542757-8b87b70751a3df0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

语法检测

![image.png](http://upload-images.jianshu.io/upload_images/1542757-fafc68a28f730042.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

#### list列出

![image.png](http://upload-images.jianshu.io/upload_images/1542757-0c56cb6847ec32dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

#### forks线程
语法:  -f FORKS, --forks=FORKS

![image.png](http://upload-images.jianshu.io/upload_images/1542757-3d0fcd139dcd2e5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

#### step交互执行

![image.png](http://upload-images.jianshu.io/upload_images/1542757-8063b8927022afff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

### ansible-playbook 角色roles 定义

#### roles使用

ansible自1.2版本引入的新特性，用于层次性，结构性的组织
playbook，roles能够根据层次型结构自动装载变量文件，tasks以及hanlders等

要使用roles只需要在playbook中使用include指令即可

简单来讲，roles就是通过分别将变量，文件，任务，模块以及处理器位置放置于单独的目录中，并可以便捷地include他们的一种机制，角色一般用于基于主机机构建服务的场景中

但也可以用于构建守护进程等场景中
> 一个roles的案例如下所示

```
site.yml
webservers.yml
fooservers.yml
roles/
       common/
              files/
              templates/
              tasks/
              handlers/
              vars/
              meta/
       webservsers/     
              files/    
              templates/
              tasks/
              handlers/
              vars/
              meta/
```
- 在playbook中，可以这样使用roles
```
---
- hosts: webservers
  roles:
    - common
    - webservers
```
- 也可以向roles传递参数
```
---
- hosts: webservsers
  roles:
    - common
    - { role: foo_app_instance，dir: '/opt/a' port: 5000 }
```
- 甚至可以条件式的使用roles
```
---
- hosts: webservers
  roles:
    - { roles: some_role, when: "ansible_os_family == 'Redhat' "}
```
- 还可以role之间设定依赖关系
 
 
 
#### roles创建步骤

-  创建以roles命名的目录
- 在roles目录中分别创建以各个角色命名的目录， 如webservers等
- 在每个角色命名的目录中分别创建files，handles，meta，tasks，templates，和vars等目录，用不到的可以创建为空目录，也可以不创建
- 在playbook文件中，调用各个角色

#### roles内各个目录中可用的文件

- tasks目录: 至少应该包含一个名为main.yml的文件，其定义了此角色的任务列表，此文件可以使用include包含其他的位于此目录中的task文件
- files目录: 存放由copy 和script等模块调用的文件
- templates目录: template模块会自动再次目录中寻找Jinja2模板文件
- handlers目录: 此目录中应当包含一个main.yml文件，用于定义此角色用到的各handler：在handler中使用include包含的其它的handler文件也应该位于此目录中
- vars目录: 应当包含一个main.yml文件，用于定义此角色用到的变量
- meta目录: 应当包含一个main.yml文件， 用于定义此角色的特殊设定的依赖关系， ansible 1.3及以后的版本才支持
- default目录： 为当前角色设定默认变量时使用的目录，应当包含一个main.yml文件
 
#### roles总结

- 目录名同为角色名
- 目录结构有固定格式
  - files：静态文件
  - templates： Jinja2 模板文件
  - tasks：至少有一个main.yml文件，定义各tasks
  - handlers： 至少有一个main.yml文件，定义各handlers
  - vars：定义变量
  - meta：定义依赖关系信息
- site.yml中定义playbook， 额外也可以有其他的yml文件
 
#### roles简单案例
 
> 建立目录

```
[root@bj-idc-14 playbooks]# mkdir -pv /root/ansible_playbooks/roles/{websrvs,dbsrvs}/{tasks,files,templates,meta,handlers,vars}
``` 
 
>查看目录树

```
[root@bj-idc-14 playbooks]# tree /root/ansible_playbooks/
/root/ansible_playbooks/
└── roles
├── dbsrvs
│   ├── files
│   ├── handlers
│   ├── meta
│   ├── tasks
│   ├── templates
│   └── vars
└── websrvs
├── files
├── handlers
├── meta
├── tasks
├── templates
└── vars
```
 
```
[root@bj-idc-14 ansible_playbooks]# cat site.yml
---
- hosts: web
  remote_user: root
  roles:
- websrvs
- hosts: web
  remote_user: root
  roles:
- dbsrvs
```
 
目录树

![image.png](http://upload-images.jianshu.io/upload_images/1542757-23cb804b91cbb76c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

执行结果
第一次websrvs

![image.png](http://upload-images.jianshu.io/upload_images/1542757-3b37901c361d26f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

第二次websrvs和dbsrvs

![image.png](http://upload-images.jianshu.io/upload_images/1542757-83550b94446b36bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)
