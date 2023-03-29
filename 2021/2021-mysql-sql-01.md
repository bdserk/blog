```
"""
create table emp( 

	id int not null unique auto_increment,
	name varchar(20) not null,
	sex enum('male','female','others') not null default 'male',
	age int(3) unsigned not null default 18,
	hire_date date not null,
	post varchar(30),
	post_comment varchar(100),
	salary double(15,2),
	office int,
	depart_id int
);
insert into emp(name,sex,age,hire_date,post,salary,office,depart_id) values 
('jason','male',18,'20170301','shuaige',7300.33,401,1),
('tom','male',78,'20170312','techer',8300.33,401,1),
('tony','male',18,'20140301','techer',3500.33,401,1),
('owen','female',18,'20150301','techer',2100.33,401,1),
('jack','male',28,'20180301','techer',9000.33,401,1),
('haha','male',38,'20170801','sale',1000.33,402,2),
('heihei','male',19,'20130401','sale',2000.28,402,2),
('anqila','female',20,'20140601','operation',19000,403,3),
('zhongwuyan','female',22,'20130401','operation',17000,403,3);
"""
```



where 约束条件

```python
select * from emp where id >= 3 and id <= 6;
select * from emp where id between 3 and 6;


select * from emp where salary=2000 or salary=18000 or salary=17000;
select * from emp where salary in(2000,17000,18000);

模糊匹配
	% 匹配多个字符
	_ 匹配单个字符
select * from emp where name like '%o%';
select * from emp where name like '__o%';


select * from emp where salary not in (2000,1800,17000);

查询岗位描述为空的 用is 
select * from emp where post_comment is NULL;
select * from emp where not salary = 17000;
```

groupby 分组

```

show variables like '%mode';
set global sql_mode = 'strict_trans_tables,only_full_group_by';


什么时候需要分组
	关键字： 每个 平均 最高 最低
select post,max(salary) from emp group by post;

方法： sum min max avg count 
select post.sum(salary) from emp group by post;

分组以后 列出每个部门的员工姓名，还可以进行拼接
select post,group_concat(name) from emp group by post;
select post,group_concat(name,'_ing') from emp group by post;
不分组的时候用
select concat('Name:',name) from emp ;

补充 as 语法不简单，可以给字段起个别名， 还可以给表起别名
select emp.id,emp.name from emp;
select t1.id ,t1.name from emp as t1; #临时有效

查询每个人的年薪
select name,salary * 12  from emp;
```

分组注意事情

```
关键字 group by 和 where： group by 必须再where 后面
where 先对整体数据进行过滤之后在分组操作
聚合函数 只能够 再分组后使用

select max(salary) from emp;  # 不分组 整个表就是一个组

统计各个部门年龄大于20岁以上的平均薪资
select post, avg(salary)  from  emp where age > 20  group by post;

```



having 分组以后的筛选

```python
"""
having的语法 跟where 是一致的
只不过 having 是再分组以后进行的过滤操作
即 having 可以直接使用聚合函数
"""

1 统计各个部门年龄大于20 以上的员工平均薪资 并且保留 平均薪资大于10000的部门
select post, avg(salary)  from  emp where age > 20  group by post having avg(salary) > 10000;

```

distinct 去重

```
注意： 必须是完全一样的数据才可以去重
select distinct age from emp;

```

order by 排序

```python 
"""
order by 默认是升序 ，asc 可以省略不写

可以修改为降序： desc 
"""
select * from emp order by salary asc;
select * from post,avg(salary) from emp
	where age > 10 group by post having avg(salary) > 1000 order by avg(salary) desc ;
  
  
```

limit 限制展示条数

```
limit 3 ;

limit 0,5;
```



正则

```
select * from emp where name regexp '^j.*(n|y)$'; 

```

连表查询

```
"""
inner join 
left join 
right join 
union join 
"""
inner join 内连接
select * from emp inner join dep on emp.dep_id = depm.id ;
left join  
左表所有的数据都展示出来 
right join
右表所有的数据都展示出来，没有对应的项就是null
union join 全连接  
左右俩个表的数据全部展示出来

```

**MySQL不区分大小写**

**建议所有的关键字 是大写的** 



class student score course teacher 五个表



select  course.cname,teacher.tname from course inner join teacher on course.teacher_id = teacher.tid; 

查询平均成绩大于八十分的同学的姓名和平均成绩



select * from student inner join t1 on student.sid = t1.course_id  where student

(select course.cid,course.cname,score.student_id  from course  inner join score on course.cid = score.course.id ) as t1 



