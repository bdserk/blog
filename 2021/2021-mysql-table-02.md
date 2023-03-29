约束条件

```python
"""
not null
zerofill
unsigned


default 
	gender enum('male','female','others') default 'male'
unique 
	单列唯一
		id int unique
	联合唯一 
		unique(id,age)
		
primary key 
	
"""
```



 





一对多的关系

````python
"""
判断表与表之间关系的时候，分别站在俩张表的角度考虑
	员工表： 
		一个员工不能对应多个部门
	
	
	部门表： 
	  一个部门可以拥有多个员工
  结论：
  	员工表和部门表属于一对多

"""

foreign key 
	1 一对多关系 外键字段在多的一方
  2 再创建表的时候， 一定要建立 被关联表
  3 在录入数据的时候， 也必须先录入 被关联表

"""  
#

create table dep(
  id int primary key auto_increment,
  dep_name char(16),
  dep_desc char(32)
);
create table emp(
  id int primary key auto_increment,
  name char(16),
  gender enum('male','female','others') default 'male',
  dep_id int,
  foreign key(dep_id) references dep(id)
  
  on update cascade  #同步更新
  on delete cascade  #同步删除
  
  insert into dep(dep_name,dep_desc) values('tech技术部','tech'),('平台架构部'，'云平台'),('财务部','公司财务')
  insert into emp('bds','1'),('bdser','2'),('budongshu','3')      
  """
````

多对多

```python
'''
create table book(
	id int primary key auto_increment,
	title varchar(32),
	price int
);
create table author(
	id int primary key auto_incremnt,
	name varchar(32),
	age int
)
create table book2author(
	id int primary key auto_increment,
	author_id int,
	book_id int,
	foreign key(author_id) references author(id)
	on update cascade 
	on delete cascade 
	foreign key(book_id) references book(id)
	on update cascade 
	on delete cascade
)

insert into book(title,price) values('jpm',100),('vim',120),('python','40')


```

一对一 

```python
一对一表关系 通过id 字段进行链接，那么id字段必须unique 唯一。
推荐健在查询频率比较高的表中
```



修改表

```python
"""
# 不区分大小写
修改表名字
alter table 表名 rename 新表名；

增加字段
alter table 表名 add 字段名 字段类型（宽度） 约束条件；
alter table 表名 add 字段名 字段类型（宽度） 约束条件； first/after 
删除字段
alter  table 表名 drop 字段名 #字段原来的数据都没有了

修改字段
alter table 表名 modify 字段名 字段类型（宽度） 约束条件
alter tbale 表名 change 旧字段名 新字段名 字段类型（宽度） 约束条件

"""

```

复制表

```python
"""
我们sql语句查询的结果起始就是一张虚拟表
create table 新表名 select * from dep;  #不能复制 外键 主键 约束
create table new_depp select * from dep where id > 300; # 如果where条件后 没有数据，表结构还是可以创建
"""

```



补充

```python
"""
表与表之间如果有关系的化 可以通过俩种简历联系的方式
1 就是通过外键强制性的建立关系

2 就是自己通过sql语句逻辑层面上建立关系
	delete from emp where id =1;
	delete from dep where  id= 1; 

因为创建外键会消耗一定的资源，并且增加了表与表之间的耦合度 
在实际项目中 如果表特别多，其实可以不做任何外键处理，直接通过sql语句来建立逻辑层面上的关系
取决于实际项目需求
"""

```



