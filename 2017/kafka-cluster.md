## kafka 集群

### 机器环境 
```
系统版本
centos6.6
机器三台搭建集群
10.1.1.11
10.1.1.12
10.1.1.13
```
*以下安装的软件将分别在三台机器进行安装,下面只用一台机器作为例子进行安装。不同的地方会有说明*


### zookeeper 安装

#### 下载并解压zk
`download 地址:` <https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/>

```
cd /home/root/
wget http://archive.apache.org/dist/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz
tar xf zookeeper-3.4.9.tar.gz
cd zookeeper-3.4.9 
```
#### 创建快照日志和日志目录
```
mkdir /Data/zookeeper/data -pv 
mkdir /Data/zookeeper/logs -pv 

```
#### 安装并配置
`安装目录: /Data/apps/`
 
```
cp -a /root/zookeeper-3.4.9 /Data/apps/zookeeper
```
`配置文件修改`

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/Data/zookeeper/data
dataLogDir=/Data/zookeeper/logs

clientPort=2181
server.1=10.1.1.11:2888:3888
server.2=10.1.1.12:2888:3888
server.3=10.1.1.13:2888:3888
```
#### zk 标识server id
`目录地址: /Data/zookeeper/data/myid` 
> 在目录中创建文件myid文件 每个文件中分别写入当前机器的server id

> 例如这个机器10.1.1.11 

```
将分别在三台机器上执行
echo 1 >>  /Data/zookeeper/data/myid
echo 2 >>  /Data/zookeeper/data/myid
echo 3 >>  /Data/zookeeper/data/myid

```
#### 启动zookeeper 

```
/Data/apps/zookeeper/bin/zkServer.sh start 
```

#### 检测状态
> 在各个节点上分别执行如下指令，可看到其中有leader和follower，即搭建成功

```
/Data/apps/zookeeper/bin/zkServer.sh status

```

### kafka安装
`DOWNLOAD 地址:` <http://kafka.apache.org/downloads.html>

#### 下载 kafka
安装目录: `/Data/apps/kafka` 

> 分别在三体机器上进行安装


```
wget https://archive.apache.org/dist/kafka/0.9.0.1/kafka_2.10-0.9.0.1.tgz
tar xf kafka_2.10-0.9.0.1.tgz
cp -a kafka_2.11-0.9.0.1 /Data/apps/kafka 

```

#### 建立日志目录

```
mkdir /Data/kafka/kafka-logs

```

#### 配置文件 kafka


```  
broker.id=0             #集群节点的标示符，不得重复。取值范围0~n
host.name=10.1.1.11     #三个机器分别修改为自己的IP地址
port=9092
zookeeper.connect=10.1.1.11:2181,10.1.1.12:2181,10.1.1.13:2181
default.replication.factor=2
num.network.threads=3
num.io.threads=8
num.partitions=1
num.recovery.threads.per.data.dir=1
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/Data/kafka/kafka-logs
log.retention.check.interval.ms=300000  
log.cleanup.policy=delete        #日志的清除策略：直接删除
log.retention.hours=72           #日志保存时间为3天
log.segment.bytes=1073741824     #每个日志文件的最大的大小，这里为1GB
delete.topic.enable=true         #通过配置此项使得删除topic的指令生效
zookeeper.connection.timeout.ms=6000
```
#### 启动 kafka
`这里注意，记得要先启动zookeeper。确保zookeeper启动以后再执行kafka的启动命令`

```
/Data/apps/kafka/bin/kafka-server-start.sh -daemon /Data/apps/kafka/config/server.properties

```
####

```
ps aux |grep kafka 

```
#### kafka 常用操作命令

##### Check service 
推荐第一次启动先不要加上`-daemon`参数 观察一下控制台输出是否有success
 
```
/Data/apps/kafka/bin/kafka-server-start.sh /Data/apps/kafka/config/server.properties
```
##### Create topic 
```
bin/kafka-topics.sh --create --zookeeper 10.1.1.11:2181 --replication-factor 3 --partitions 1 --topic bdstest
```
##### Describe a topic

```
bin/kafka-topics.sh --describe --zookeeper 10.1.1.11:2181 --topic bdstest
```

##### List the topic
```
bin/kafka-console-producer.sh --broker-list 10.1.1.11:9092 --topic bdstest
```
##### Start a consumer
```
bin/kafka-console-consumer.sh --zookeeper 10.1.1.11:2181 --topic bdstest --from-beginning
```
##### Start a consumer
```
bin/kafka-console-consumer.sh --zookeeper 10.1.1.12:2181 --topic nodeHlsTest --from-beginning
```
#####  Delete a topic
要事先在`serve.properties` 配置 `delete.topic.enable=true`

```
bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic bdstest
```
**如果仍然只是仅仅被标记了删除(zk中并没有被删除)，那么启动zkCli.sh,输入如下指令**

```
/Data/apps/zookeeper/bin/zkCli.sh   #敲回车进入zookeeper管理终端
[zk: localhost:2181(CONNECTED) 0] ls /brokers/topics
[test1, test2]
[zk: localhost:2181(CONNECTED) 1] rmr /brokers/topics/test1 

```










