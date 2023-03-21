
---
title: filebeat config

date: 2017-06-11 03:22:31

categories: elk

tags: filebeat

---

### Filebeat Prospector

```
filebeat.prospectors:
- input_type: log
  paths:
    - /var/log/apache/httpd-*.log
  document_type: apache

- input_type: log
  paths:
    - /var/log/messages
    - /var/log/*.log

```

### Filebeat Options

#### input_type: log|stdin   	

* 指定输入类型

#### paths   

* 支持基本的正则，所有golang glob都支持,支持/var/log/\*/*.log 

#### encoding 
	
- plain, latin1, utf-8, utf-16be-bom, utf-16be, utf-16le, big5, gb18030, gbk, hz-gb-2312,
- euc-kr, euc-jp, iso-2022-jp, shift-jis, and so on 

#### exclude_lines  

* 支持正则 排除匹配的行，如果有多行，合并成一个单一行来进行过滤

#### include\_lines 

* 支持正则 include\_lines执行完毕之后会执行exclude_lines。

#### exclude_files  

- 支持正则 排除匹配的文件 
- exclude_files: ['\.gz$']

#### tags  


* 列表中添加标签，用过过滤

	```
		filebeat.prospectors:
		- paths: ["/var/log/app/*.json"]
  		  tags: ["json"]
  	```

#### fields 

* 可选字段，选择额外的字段进行输出 
* 可以是标量值，元组，字典等嵌套类型 
* 默认在sub-dictionary 位置  

	```
		filebeat.prospectors:	
		- paths: ["/var/log/app/*.log"]
     	  fields:
      	    app_id: query_engine_12	
	
	```
	
#### fields_under\_root 

- 如果值为ture，那么fields存储在输出文档的顶级位置 
- 如果与filebeat中字段冲突，自定义字段会覆盖其他字段
	
	```
	fields_under_root: true
	fields:
  	  instance_id: i-10a64379
  	  region: us-east-1
	
	```
	
#### ignore_older 

* 可以指定Filebeat忽略指定时间段以外修改的日志内容 
* 文件被忽略之前，确保文件不在被读取，必须设置ignore older时间范围大于close_inactive 
* 如果一个文件正在读取时候被设置忽略，它会取得到close_inactive后关闭文件，然后文件被忽略

#### close_* 

>  close_ *配置选项用于在特定标准或时间之后关闭harvester。 关闭harvester意味着关闭文件处理程序。 如果在harvester关闭后文件被更新，则在scan_frequency过后，文件将被重新拾取。 但是，如果在harvester关闭时移动或删除文件，Filebeat将无法再次接收文件，并且harvester未读取的任何数据都将丢失。

#### close_inactive 

* 启动选项时，如果在制定时间没有被读取，将关闭文件句柄 
* 读取的最后一条日志定义为下一次读取的起始点，而不是基于文件的修改时间 
* 如果关闭的文件发生变化，一个新的harverster将在scan_frequency运行后被启动 
* 建议至少设置一个大于读取日志频率的值，配置多个prospector来实现针对不同更新速度的日志文件
* 使用内部时间戳机制，来反映记录日志的读取，每次读取到最后一行日志时开始倒计时
* 使用2h 5m 来表示 

#### close_rename 

* 当选项启动，如果文件被重命名和移动，filebeat关闭文件的处理读取

#### close_removed 

* 当选项启动，文件被删除时，filebeat关闭文件的处理读取 
* 这个选项启动后，必须启动clean_removed  

#### close_eof 	

* 适合只写一次日志的文件，然后filebeat关闭文件的处理读取 

#### close_timeout 

* 当选项启动时，filebeat会给每个harvester设置预定义时间，不管这个文件是否被读取，达到设定时间后，将被关闭 
* close_timeout 不能等于ignore_older,会导致文件更新时，不会被读取
* 如果output一直没有输出日志事件，这个timeout是不会被启动的，至少要要有一个事件发送，然后haverter将被关闭
* 设置0 表示不启动 

#### clean_inactived 

- 从注册表文件中删除先前收获的文件的状态
- 设置必须大于ignore_older+scan_frequency，以确保在文件仍在收集时没有删除任何状态
- 配置选项有助于减小注册表文件的大小，特别是如果每天都生成大量的新文件
- 此配置选项也可用于防止在Linux上重用inode的Filebeat问题

#### clean_removed 

* 启动选项后，如果文件在磁盘上找不到，将从注册表中清除filebeat
* 如果关闭close removed 必须关闭clean removed 

#### scan_frequency 

- prospector检查指定用于收获的路径中的新文件的频率,默认10s	

#### document_type 

* 类型事件，被用于设置输出文档的type字段，默认是log

#### harvester_buffer_size 

* 每次harvester读取文件缓冲字节数，默认是16384 

#### max_bytes 

* 对于多行日志信息，很有用，最大字节数 

#### json 
> 这些选项使Filebeat解码日志结构化为JSON消息 
  逐行进行解码json 
  
- keys_under_root
	- 设置key为输出文档的顶级目录 
- overwrite_keys 
	- 覆盖其他字段
- add\_error_key
	- 定一个json_error 
- message_key 
	- 指定json 关键建作为过滤和多行设置，与之关联的值必须是string 
		
#### multiline	

控制filebeat如何处理跨多行日志的选项，多行日志通常发生在java堆栈中 

`multiline.pattern: '^\['`
`multiline.negate: true`
`multiline.match: after`
	
上面匹配是将多行日志所有不是以[符号开头的行合并成一行它可以将下面的多行日志进行合并成一行

```
[beat-logstash-some-name-832-2015.11.28] IndexNotFoundException[no such index]
    at org.elasticsearch.cluster.metadata.IndexNameExpressionResolver$WildcardExpressionResolver.resolve(IndexNameExpressionResolver.java:566)
    at org.elasticsearch.cluster.metadata.IndexNameExpressionResolver.concreteIndices(IndexNameExpressionResolver.java:133)
    at org.elasticsearch.cluster.metadata.IndexNameExpressionResolver.concreteIndices(IndexNameExpressionResolver.java:77)
    at org.elasticsearch.action.admin.indices.delete.TransportDeleteIndexAction.checkBlock(TransportDeleteIndexAction.java:75)

```

#### multiline.pattern 

- 指定匹配的正则表达式，filebeat支持的regexp模式与logstash支持的模式有所不同
	[pattern regexp](https://www.elastic.co/guide/en/beats/filebeat/current/regexp-support.html)

#### multiline.negate 

- 定义上面的模式匹配条件的动作是 否定的，默认是false 
- 假如模式匹配条件'^b'，默认是false模式，表示讲按照模式匹配进行匹配 将不是以b开头的日志行进行合并
- 如果是true，表示将不以b开头的日志行进行合并

#### multiline.match 

- 指定Filebeat如何将匹配行组合成事件,在之前或者之后，取决于上面所指定的negate 
	
#### multiline.max_lines

- 可以组合成一个事件的最大行数，超过将丢弃，默认500 

#### multiline.timeout 

- 定义超时时间，如果开始一个新的事件在超时时间内没有发现匹配，也将发送日志，默认是5s

#### tail_files

- 如果此选项设置为true，Filebeat将在每个文件的末尾开始读取新文件，而不是开头 
- 此选项适用于Filebeat尚未处理的文件

#### symlinks

- 符号链接选项允许Filebeat除常规文件外,可以收集符号链接。收集符号链接时，即使报告了符号链接的路径，Filebeat也会打开并读取原始文件。

#### backoff 

- backoff选项指定Filebeat如何积极地抓取新文件进行更新。默认1s 
- backoff选项定义Filebeat在达到EOF之后再次检查文件之间等待的时间。

#### max_backoff

- 在达到EOF之后再次检查文件之前Filebeat等待的最长时间

#### backoff_factor

- 指定backoff尝试等待时间几次，默认是2

#### harvester_limit 

- harvester_limit选项限制一个prospector并行启动的harvester数量，直接影响文件打开数 
	
#### enabled

- 控制prospector的启动和关闭

### filebeat global 

#### spool_size 
	
- 事件发送的阀值，超过阀值，强制刷新网络连接

	```
	filebeat.spool_size: 2048
	```

#### publish_async 

- 异步发送事件，实验性功能

#### idle_timeout 

- 事件发送的超时时间，即使没有超过阀值，也会强制刷新网络连接

	```
	filebeat.idle_timeout: 5s
	```

#### registry_file 

- 注册表文件的名称，如果使用相对路径，则被认为是相对于数据路径 
- 有关详细信息，请参阅目录布局部分 默认值为${path.data}/registry
   
   ```
	filebeat.registry_file: registry
	```
	
#### config_dir 

- 包含额外的prospector配置文件的目录的完整路径 
- 每个配置文件必须以.yml结尾 
- 每个配置文件也必须指定完整的Filebeat配置层次结构，即使只处理文件的prospector部分。
- 所有全局选项（如spool_size）将被忽略 
- 必须是绝对路径
	
	```
	filebeat.config_dir: path/to/configs
	```
	
#### shutdown_timeout 

- Filebeat等待发布者在Filebeat关闭之前完成发送事件的时间。 

### Filebeat General 
#### name 

- 设置名字，如果配置为空，则用该服务器的主机名
	
	```
	name: "my-shipper"	
	```
	
#### queue_size

- 单个事件内部队列的长度 默认1000
 
#### bulk\_queue_size 

- 批量事件内部队列的长度 

#### max_procs

- 设置最大使用cpu数量

#### geoip.paths 

- 此配置选项目前仅由Packetbeat使用，它将在6.0版中删除
- 要使GeoIP支持功能正常，GeoLite City数据库是必需的。

	```
    geoip:
      paths:
        - "/usr/share/GeoIP/GeoLiteCity.dat"
        - "/usr/local/var/GeoIP/GeoLiteCity.dat"
	
	```
	
### Filebeat reload 
> 属于测试功能 
	
#### path 

- 定义要检查的配置路径		

#### reload.enabled 

- 设置为true时，启用动态配置重新加载。	

#### reload.period	

- 定义要检查的间隔时间 	

```
filebeat.config.prospectors:
  path: configs/*.yml
  reload.enabled: true
  reload.period: 10s

```
