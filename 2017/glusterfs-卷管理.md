## 分布式存储glusterfs 卷类型管理
- 扩展卷
- 收缩卷
- 平衡卷

### 扩展卷
可以在线扩展卷的容量
可以加一个brick到分布卷 增加分布式卷的容量
可以增加一组brick到分布式复制卷，增加卷的容量
### 收缩卷
可以在线收缩
当一个brick变得不可用时，可以使用该命令来删除brick 
当删除时，只是该brick的配置信息被删除，你可以继续直接访问brick中的数据
### 平衡卷
你通过添加brick和删除brick进行扩容和收缩卷后，需要在服务器之间重新平衡数据
一般平衡数据有如下两种场景
- Fix Layout：重新定位layout，原来的layout不变，（新）数据写入新的节点
- Fix Layout and Migrate Data:重新定位layout的修改，并且迁移已经存在的数据

### 分布式卷的管理
```
# 扩展卷 
gluster volume add-brick gv2 c6-vm1:/data/gfs2
# 平衡卷
gluster volume rebalance gv2 start 
gluster volume rebalance gv2 status
# 收缩卷
gluster volume remove-brick gv2 c6-vm1:/data/gfs2 start 
gluster volume remove-brick gv2 c6-vm1:/data/gfs2 status
gluster volume remove-brick gv2 c6-vm1:/data/gfs2 commit 
```
### 复制卷的管理

##### 扩展卷 
```
gluster volume add-brick gv2  replica 4  c6-vm1:/data/gfs1 c6-vm1:/data/gfs2
```

##### 平衡卷
```
### 平衡布局
gluster volume rebalance gv2
### 平衡数据
gluster volume rebalance gv2 start 
gluster volume rebalance gv2 status
### 停止rebalance
gluster volume stop test-volume     
```

##### 收缩卷
```
gluster volume remove-brick gv2  replica 2 c6-vm1:/data/gfs2 c6-vm2:/data/gfs2  start 
gluster volume remove-brick gv2  relica 2  c6-vm1:/data/gfs2 c6-vm2:/data/gfs2 status
gluster volume remove-brick gv2  relica 2  c6-vm1:/data/gfs2 c6-vm2:/data/gfs2commit 
```



### 分布式复制卷的管理

##### 扩展卷
```
# 如果是复制卷 replica后面的数字是增加brick后 总的数量 
# 原来这里有俩个 我在增加俩个 所以总共是4个
[root@c6-vm1 data]# gluster volume add-brick gv2 replica 2 c6-vm1:/data/gfs2 c6-vm2:/data/gfs2 force
volume add-brick: success
[root@c6-vm1 data]# gluster volume info

Volume Name: gv2
Type: Distributed-Replicate
Volume ID: fa64409f-814b-4191-b9e4-8c317bf63a94
Status: Started
Snapshot Count: 0
Number of Bricks: 2 x 2 = 4
Transport-type: tcp
Bricks:
Brick1: c6-vm1:/data/gfs1
Brick2: c6-vm2:/data/gfs1
Brick3: c6-vm1:/data/gfs2
Brick4: c6-vm2:/data/gfs2
Options Reconfigured:
transport.address-family: inet
nfs.disable: off
```
在client写入新数据
```
[root@c6-vm3 mnt]# ll
总用量 21
drwxr-xr-x 2 root root 4096 10月 10 2018 dir1
drwxr-xr-x 2 root root 4096 10月 10 2018 dir2
drwxr-xr-x 2 root root 4096 10月 10 2018 dir3
drwxr-xr-x 2 root root 4096 10月 11 2018 dir4
-rw-r--r-- 1 root root    0 10月  2 12:54 file1
-rw-r--r-- 1 root root    0 10月  2 12:54 file10
-rw-r--r-- 1 root root    5 10月  9 2018 file2
-rw-r--r-- 1 root root    0 10月  2 12:54 file3
-rw-r--r-- 1 root root    0 10月  2 12:54 file4
-rw-r--r-- 1 root root    0 10月  2 12:54 file5
-rw-r--r-- 1 root root    0 10月  2 12:54 file6
-rw-r--r-- 1 root root    0 10月  2 12:54 file7
-rw-r--r-- 1 root root    0 10月  2 12:54 file8
-rw-r--r-- 1 root root    0 10月  2 12:54 file9
drwx------ 2 root root 4096 10月  9 2018 lost+found
[root@c6-vm3 mnt]# mkdir dir5
[root@c6-vm3 mnt]# cd dir5
[root@c6-vm3 dir5]# ls
[root@c6-vm3 dir5]# dd if=/dev/zero of=bds13 bs=1024 count=10000
记录了10000+0 的读入
记录了10000+0 的写出
10240000字节(10 MB)已复制，0.792822 秒，12.9 MB/秒
[root@c6-vm3 dir5]# dd if=/dev/zero of=bds14 bs=1024 count=10000
记录了10000+0 的读入
记录了10000+0 的写出
10240000字节(10 MB)已复制，0.795706 秒，12.9 MB/秒
```
去存储端看存储目录信息
可以看到只是把目录信息同步过来了 ，新数据并没有写入新增的磁盘中
```
[root@c6-vm1 gfs1]# tree
.
├── dir1
│   ├── bds1
│   ├── bds2
│   └── bds3
├── dir2
│   ├── bds4
│   ├── bds5
│   └── bds6
├── dir3
│   ├── bds7
│   ├── bds8
│   └── bds9
├── dir4
│   ├── bds10
│   ├── bds11
│   └── bds12
├── dir5
│   ├── bds13
│   └── bds14
├── file1
├── file10
├── file2
├── file3
... ... 

6 directories, 24 files
[root@c6-vm1 gfs1]# cd ../gfs2
[root@c6-vm1 gfs2]# ls
dir1  dir2  dir3  dir4  dir5  lost+found
[root@c6-vm1 gfs2]# tree
.
├── dir1
├── dir2
├── dir3
├── dir4
├── dir5
└── lost+found

6 directories, 0 files
```
##### 平衡卷
```
[root@c6-vm1 gfs2]# gluster volume rebalance gv2 start
volume rebalance: gv2: success: Rebalance on gv2 has been started successfully. Use rebalance status command to check status of the rebalance process.
ID: 4a4e046c-46f9-46c3-9010-f359e5cd04c2
[root@c6-vm1 gfs2]# gluster volume rebalance gv2 status
                                    Node Rebalanced-files          size       scanned      failures       skipped               status  run time in h:m:s
                               ---------      -----------   -----------   -----------   -----------   -----------         ------------     --------------
                               localhost                0        0Bytes            24             0             6            completed        0:00:00
                                  c6-vm2                0        0Bytes             0             0             0            completed        0:00:00
volume rebalance: gv2: success
[root@c6-vm1 gfs2]# ll
总用量 36
drwxr-xr-x 2 root root 4096 10月 11 09:58 dir1
drwxr-xr-x 2 root root 4096 10月 11 09:58 dir2
drwxr-xr-x 2 root root 4096 10月 11 09:58 dir3
drwxr-xr-x 2 root root 4096 10月 11 09:46 dir4
drwxr-xr-x 2 root root 4096 10月 11 09:47 dir5
---------T 2 root root    0 10月 11 09:58 file5
---------T 2 root root    0 10月 11 09:58 file6
---------T 2 root root    0 10月 11 09:58 file8
drwx------ 2 root root 4096 10月  9 03:12 lost+found
[root@c6-vm1 gfs2]# tree
.
├── dir1
│   └── bds2
├── dir2
│   └── bds4
├── dir3
│   └── bds7
├── dir4
├── dir5
├── file5
├── file6
├── file8
└── lost+found

6 directories, 6 files
```
我这里再次在client端创建新数据 看看会不会落到新存储节点
```
[root@c6-vm3 ~]# cd /mnt
[root@c6-vm3 mnt]# mkdir dir7
[root@c6-vm3 mnt]# cd dir7
[root@c6-vm3 dir7]# dd if=/dev/zero of=bds17  bs=1024 count=10000
记录了10000+0 的读入
记录了10000+0 的写出
10240000字节(10 MB)已复制，0.819898 秒，12.5 MB/秒
[root@c6-vm3 dir7]# dd if=/dev/zero of=bds18  bs=1024 count=10000
记录了10000+0 的读入
记录了10000+0 的写出
10240000字节(10 MB)已复制，0.966085 秒，10.6 MB/秒
```
需要注意几个地方
1 可以看到新数据已经落到新加入节点上
2 平衡卷有俩种

   - 平衡布局相当于重新分配了hash范围，从而使新数据写入新节点
       - gluster volume rebalance gv2

   - 平衡数据
       - gluster volume rebalance gv2 start 
       - gluster volume rebalance gv2 status

```
[root@c6-vm1 gfs2]# tree
.
├── dir1
│   └── bds2
├── dir2
│   └── bds4
├── dir3
│   └── bds7
├── dir4
├── dir5
├── dir6
│   └── bds15
├── dir7
│   └── bds17
├── file5
├── file6
├── file8
└── lost+found

8 directories, 8 files
[root@c6-vm1 gfs2]# du -sh ./*
8.0K	./dir1
8.0K	./dir2
8.0K	./dir3
4.0K	./dir4
4.0K	./dir5
9.8M	./dir6
9.8M	./dir7
4.0K	./file5
4.0K	./file6
4.0K	./file8
4.0K	./lost+found
[root@c6-vm1 gfs2]# cd dir1
[root@c6-vm1 dir1]# ls
bds2
### 可以看到文件权限是大写T
[root@c6-vm1 dir1]# ll -lh
总用量 4.0K
---------T 2 root root 0 10月 11 09:58 bds2
[root@c6-vm1 dir1]# cd ..
[root@c6-vm1 gfs2]# ll
```
##### 收缩卷
收缩卷前gluster需要先移动数据到其他位置
刚才我们新增加的数据 有个俩个10M数据写入节点
可以看到下面rebalanced-file 是2，size 是19.5MB 

```
[root@c6-vm1 data]#  gluster volume remove-brick gv2 replica 2  c6-vm1:/data/gfs2 c6-vm2:/data/gfs2 start
volume remove-brick start: success
ID: c72ae686-b12f-4652-b007-d719f82130c0
[root@c6-vm1 data]#  gluster volume remove-brick gv2 replica 2  c6-vm1:/data/gfs2 c6-vm2:/data/gfs2 status
                                    Node Rebalanced-files          size       scanned      failures       skipped               status  run time in h:m:s
                               ---------      -----------   -----------   -----------   -----------   -----------         ------------     --------------
                               localhost                2        19.5MB            34             0             0            completed        0:00:01
                                  c6-vm2                0        0Bytes             0             0             0            completed        0:00:00
[root@c6-vm1 gfs1]#  gluster volume remove-brick gv2 replica 2  c6-vm1:/data/gfs2 c6-vm2:/data/gfs2 commit
Removing brick(s) can result in data loss. Do you want to Continue? (y/n) y
volume remove-brick commit: success
Check the removed bricks to ensure all files are migrated.
If files with data are found on the brick path, copy them via a gluster mount point before re-purposing the removed brick.                                  
```
##### 删除卷
如果想要删除卷，可以采用下面的命令：
```
#移除挂载
umount /mnt   
gluster volume stop gv2
gluster volume delete gv2
gluster volume info gv2
```

##### 迁移卷
未完成






