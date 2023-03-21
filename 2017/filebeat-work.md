*这个话题，你学习了解Filebeat关键构件
他们之间是怎么样工作， 理解这些概念，
在配置filebeat的时候，帮助你做出正确的决定。*

**filebeat有俩个主要的组件： prospectors or harversters** 

### Harvester
- 负责读取一个单个文件的内容 
- 逐行读取一个文件，并发送内容到输出端
- 从文件的开始读取，通过文件描述符控制关闭和打开文件 
- 如果文件被移走或者重命名，依然会读取该文件
- 默认情况下,Filebeat保持打开该文件,直到到达close_inactive


#### 关闭harverter会有一下结果 

- 如果之前正在读取一个删除的日志文件，此时关闭harverter，则这个文件将被关闭，释放潜在的资源 
- scan_frequency运行后，将被开始读取文件 
- 当文件被删除或者移走，关闭harvester，文件不在被读取
- 控制harvester的关闭，使用close_* 选项配置 

### Prospector
- 负责管理harvesters和寻找资源来读取 
- 当input_type是log的时候，它将发现所有匹配定义的路径文件，为每个文件开始启动一个harvester，每个prospector都跑自己的例程
- 支持俩中prospector类型： log and stdin 
- 每个prospector类型可以被定义多次 
- 对每个文件检查是否需要启动harvester，是否已经启动，是否被忽略等定义 
- 如果harvester是关闭的，文件的尺寸改变了，新行只是被收录进来

#### 保持文件状态

- 保持文件状态，频繁写入registry file中 
- 文件状态记录最后读取文件的偏移量，确保所有日志文件被发送
- 跟踪发送的最后一行日志，继续阅读日志，直到输出端恢复正常
- filebeat运行的时候，每个prospector状态信息被保存在内存中
- filebeat被重启，将读取registry文件来恢复文件状态，确认位置
- 为了发现每个prospertor保持文件状态，对于每个文件，filebeat将存储唯一验证号来探测一个文件是否被获取了

####  保证至少一次成功投递

- filebeat为每个事件存储投递状态到registry file中
- 在定义的输出被阻塞的情况下并没有证实所有事件,Filebeat将试图发送事件,直到输出承认已收到事件
- 如果在发送事件中，filebeat关闭，它不会等待事件成功投递返回结果，当再次启动filebeat的时候，会再次发送一次事件，确保至少事件有一次被发送，但也可以在结束的时候，发送俩条事件，或者可以指定等待超时时间 通过shutdown_timeout

#### 限制

- 当输出端不可达的时候，文件被删除了，数据可能会丢失
- 写入磁盘的速度一定要大雨filebeat进程的速度 
- linux上还有可能inode重用，filebeat会跳过inode重用文件


 
