

---
title: filebeat Output

date: 2017-06-11 05:22:31

categories: elk

tags: filebeat

---
Filbeat OutPut 

<!-- more -->

### Filebeat Elasticsearch OutPut
> configure 

```
output.elasticsearch:
  hosts: ["http://localhost:9200"]
  template.enabled: true
  template.path: "filebeat.template.json"
  template.overwrite: false
  index: "filebeat"
  ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]
  ssl.certificate: "/etc/pki/client/cert.pem"
  ssl.key: "/etc/pki/client/cert.key"

```

> 定义ip port add portocalhttps 

```
output.elasticsearch:
  hosts: ["localhost"]
  protocol: "https"
  username: "admin"
  password: "s3cr3t"

```

- compression_level 
	- gzip 压缩级别 range（1-9） 
- work 
	- 发送事件到es的工作数量 
- index 
	- The index name to write events to. 
	- The default is "filebeat-%{+yyyy.MM.dd}" (for example,"filebeat-2015.04.26").

- indices
> 支持条件，基于格式字符串的字段访问和名称映射的索引选择器规则的数组 

	- index 
		- 要使用的索引格式字符串。 如果使用的字段丢失，则规则失败。 
	- mapping 
		- 映射字典为新名字
	- default 
		- 默认字符串数值
	- when 	
	 	- 选择匹配条件
	
	```
	output.elasticsearch:
	  hosts: ["http://localhost:9200"]
	  index: "logs-%{+yyyy.MM.dd}"
	  indices:
	    - index: "critical-%{+yyyy.MM.dd}"
	      when.contains:
	        message: "CRITICAL"
	    - index: "error-%{+yyyy.MM.dd}"
	      when.contains:
	        message: "ERR"
	
	```

- pipeline 
	- 格式字符串指定获取节点写入事件pipeline的id值
	
			output.elasticsearch:
  			  hosts: ["http://localhost:9200"]
  			  pipeline: my_pipeline_id
  		
	

- pipelines 

	```
	filebeat.prospectors:
	- paths: ["/var/log/app/normal/*.log"]
  		fields:
    	type: "normal"
	- paths: ["/var/log/app/critical/*.log"]
  		fields:
    	type: "critical"

	output.elasticsearch:
  	hosts: ["http://localhost:9200"]
  	index: "filebeat-%{+yyyy.MM.dd}"
  	pipelines:
	  - pipeline: critical_pipeline
		   when.equals:
        	  type: "critical"
     - pipeline: normal_pipeline
      	   when.equals:
        	  type: "normal"
	
	```

- template


		output.elasticsearch:
  		  hosts: ["localhost:9200"]
  		  template.name: "filebeat"
  		  template.path: "filebeat.template.json"
  		  template.overwrite: false
  	
	
- templates.versions 

```
output.elasticsearch:
  hosts: ["localhost:9200"]
  template.path: "filebeat.template.json"
  template.overwrite: false
  template.versions.2x.path: "filebeat.template-es2x.json

```

- max_retries 
	- 当发送失败的时候，尝试多少次发送事件 
- bulk\_max_size 
	- 单个Elasticsearch批量API索引请求中批量的最大事件数 默认值为50
- timeout 
	- The http request timeout in seconds for the Elasticsearch request 
	- The default is 90 

- flush_interval 
	- 在两个批量API索引请求之间等待新事件的秒数
- ssl 
	- <https://www.elastic.co/guide/en/beats/filebeat/current/configuration-output-ssl.html>


### Filebeat Logstash OutPut
> 需要logstash服务端安装beat插件 使用lumberjack协议发送事件到logstash 

```
output.logstash:
  hosts: ["localhost:5044"]

```

- Metadata Fields 
	- @meatedata 
		- Filebeat使用@metadata字段将元数据发送到Logstash  
		- @metadata字段的内容只存在于Logstash中，不属于从Logstash发送的任何事件的一部分
		- 有关@metadata字段的更多信息，请参阅Logstash文档[logstash doucument](https://www.elastic.co/guide/en/logstash/5.4/event-dependent-configuration.html#metadata)

				{
    				...
   				 	"@metadata": { 
      		     	  "beat": "filebeat", 
      				  "type": "<event type>" 
    				}	
				}

	 
> logstash to elasticsearch 


```
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}" 
    document_type => "%{[@metadata][type]}" 
  }
}

```

- enabled 
- hosts 
- compression_level 
- worker 
- loadbalance 
	
	```
	output.logstash:
      hosts: ["localhost:5044", "localhost:5045"]
      loadbalance: true
      index: filebeat
	```

- pipelining 
	- 异步处理事件，默认关闭	
- index
- ssl
- timeout 
- max_retries 
- bulk\_max_size 
	- The maximum number of events to bulk in a single Logstash request 
	- The default is 2048.

### kafka OutPut
> The Kafka output sends the events to Apache Kafka.

```
output.kafka:
  # initial brokers for reading cluster metadata
  hosts: ["kafka1:9092", "kafka2:9092", "kafka3:9092"]

  # message topic selection + partitioning
  topic: '%{[type]}'
  partition.round_robin:
    reachable_only: false

  required_acks: 1
  compression: gzip
  max_message_bytes: 1000000

```

- enabled 
- hosts
- version
- username 
- password 
- topic 
	
- topics 
	- topic 
	- mapping 
	- default 
	- when 

- partition 
	- random.group_events 
	- round\_robin.group_events 
	- hash.hash
	- hash.random
	
- client_id
- worker
- codec 

- metadata 
	- refresh_frequency 
	- retry.max 
	- retry.backoff 
	
- max_retries
- bulk_max_size
- timeout
- broker_timeout 
- channel\_buffer_size 
- keep_alive
- compression 
- max\_message_bytes 
- required_acks
- flush_interval	 
- ssl

### Redis OutPut 
> This output works with Redis 3.2.4.

```
output.redis:
  hosts: ["localhost"]
  password: "my_password"
  key: "filebeat"
  db: 0
  timeout: 5

```

- enabled
- hosts 
- port 
- index
- key
	
	
		output.redis:
    	  hosts: ["localhost"]
          key: "%{[fields.list]:fallback}"	
- keys 
	- key 
	- mapping 
	- default 
	- when 
	 
			output.redis:
  			  hosts: ["localhost"]
  		      key: "default_list"
  			  keys:
    	     	- key: "info_list"   # send to info_list if `message` field contains INFO
      		      when.contains:
        	        message: "INFO"
    		   - key: "debug_list"  # send to debug_list if `message` field contains DEBUG
      		     when.contains:
        	       message: "DEBUG"
    		   - key: "%{[type]}"
      			  mapping:
        			"http": "frontend_list"
        			"nginx": "frontend_list"
        			"mysql": "backend_list"
	
- passport 
- db 
- datatype 
> 
用于发布事件的Redis数据类型。如果数据类型为列表，则使用Redis RPUSH命令，并将所有事件添加到列表中，并在键下定义键。 如果使用数据类型通道，则使用Redis PUBLISH命令，这意味着所有事件都被推送到Redis的pub / sub机制。 通道的名称是键下定义的。 默认值为列表。

- codec 
- worker 
- loadbalance 
- timeout
- max_retries
- bulk_max_size 
- ssl 
- proxy_url 
- proxy_use\_local_resolver 

### File OutPut 
> 文件输出将事务转储到每个事务处于JSON格式的文件中。 目前，该输出用于测试，但可以作为Logstash的输入

```
output.file:
  path: "/tmp/filebeat"
  filename: filebeat
  #rotate_every_kb: 10000
  #number_of_files: 7
```

- enables 
- path 
	- 定义保存文件的路径
- filename 
	- 定义保存文件的名字
- rotate_every_kb 
	- 定义每个文件达到多少kb就开始切割
- number_if_files
	- 定义保存几份文件 
- codec

### Console OutPut 

```
output.console:
  pretty: true
```

- pretty 
- codec
- enabled 
- bulk\_max_size

### Codec Output

```
output.console:
  codec.json:
    pretty: true
```

```
output.console:
  codec.format:
    string: '%{[@timestamp]} %{[message]}'
```

### Loggin OutPut 

```
logging.level: warning
logging.to_files: true
logging.to_syslog: false
logging.files:
  path: /var/log/mybeat
  name: mybeat.log
  rotateeverybytes: 10MB 
  keepfiles: 7
```

### Debugging
By default, Filebeat sends all its output to syslog. When you run Filebeat in the foreground, you can use the -e command line flag to redirect the output to standard error instead. For example:

```
filebeat -e
```

The default configuration file is filebeat.yml (the location of the file varies by platform). You can use a different configuration file by specifying 
the -c flag. For example:

```
filebeat -e -c myfilebeatconfig.yml
```

You can increase the verbosity of debug messages by enabling one or more debug selectors. For example, to view the published transactions, you can start Filebeat with the publish selector like this:

```
filebeat -e -d "publish"
```

If you want all the debugging output (fair warning, it’s quite a lot), you can use *, like this:

```
filebeat -e -d "*"
```


### support
<https://www.elastic.co/support/matrix#show_compatibility>
