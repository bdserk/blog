## glusterfs 优化和命令

### gluster文件系统优化

参数项目 说明： 缺省值 合法值
```
Auth.allow IP允许授权 *(allow all) IP地址
Cluster.min-free-disk 剩余磁盘空间阙值 10% 百分比
Cluster.stripe-block-size 条带大小 128KB 字节
Network.frame-timeout 请求等待时间 1800s 0-1800
Network.ping-timeout 客户端等待时间 42s 0-42
Nfs.disabled 关闭NFS服务 On Off|on
Performance.io-thread-count IO线程数 16 0-65
Performance.cache-refresh-timepit 缓存检验周期 1s 0-61
Performace.cache-size 读缓存大小 32M 字节 
```
调整方法
```
#gluster volume set <卷> <参数>
完成之后 
#gluster volume info gv2 可查看
gv2开启预读
#gluster volume set gv2 performance.read-ahead on
gv2设置读缓存
#gluster volume set gv2 performance.cache-size 256MB
``` 
### 监控及日常维护常用命令（分布式复制）

- gluster peer status (查看当前节点和其他节点之间连接状态)
- gluster volume status gv2 (查看卷的状态)
- gluster volume status gv2 (查看这个节点有没有在线)
- gluster volume heal gv2 full (启动完全修复， 分布式复制卷会自动操作修复)
- gluster volume heal gv2 info (查看需要修复的文件)
- gluster volume heal gv2 info healed (查看修复成功的文件)
- gluster volume heal gv2 info heal-failed (查看修复失败的文件)
- gluster volume heal gv2 info split-brain (查看脑裂文件)

### 磁盘限额功能
`注意， 此功能必须到gv2目录下执行并不是 /gv2/data`

- gluster volume quota gv2 enable (激活quota功能)
- gluster volume quota gv2 disable (关闭quota功能)
- gluster volume quota gv2 limit-usage /data 10GB (/gv2/data 目录大小限制)
- gluster volum quota gv2 list (查看目录限制信息)
- gluster volume quota gv2 remove /data (删除某个目录的quota设置)
