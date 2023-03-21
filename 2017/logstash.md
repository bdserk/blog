## logstash 

> 它一个有jruby语言编写的运行在java虚拟机上的具有收集
分析转发数据流功能的工具


- 能集中处理各种类型的数据
- 能标准化不通模式和格式的数据
- 能快速的扩展自定义日志的格式
- 能非常方便的添加插件来自定义数据

### 安装logstash

- 安装jdk 
- rpm包安装

### logstash运行参数
- -f 制定配置文件，目录或者通配符加载配置信息 
- -e 用于指定字符串输入
- -w 指定filterworkers的数量，指定logstash工作线程数量
- -l 指定logstash默认日志写入文件中，默认是控制台输出
- --quiet   静默模式 仅仅只有error级别log输出
- --verbose info级别的log输出
- --debug debug级别日志的log输出
- --V 查看logstash版本
- -p  可以写自己的插件，然后指定好路径使用她们
- -t  测试logstash读取到的配置文件语法能否正常解析

### 配置语法

- input 
- filter 
- output

### 语法格式 

- 区域
	- 用{}定义区域
	- 一个区域可以定义多个插件 
- 数据类型
	- boolen： 布尔   a => true
	- Bytes：  字节   a => "10MiB"
	- Strings：字符串 a => "hello world" 
	- Number： 数值   a => 1024
	- Array：  数组   match => ["datatime","UNIX","ISO8601"]
	- Hash：   哈希   options => { key1 => "value1",key2 => "value2" }
	- 编码解码： codec: codec => "json"
	- 密码型：   my_passwd => "password"
	- 路径： 	  my_path  => "/tmp/logstash"
	- 注释：     # 
- 条件判断
	- ==,!= ,< ,> ,<= ,>= 
	- =~ 
	- in,not in 
	- and ,or , nand, xor
	- (), !() 
	- if expression { 
		} else if expression { 
			...
		} else { 
			...	
		} 		
- 字段引用
	- %{[response][status]} 


### logstash插件
- inputs   输入
- codecs	 解码
- filters  过滤 
- outputs  输出

[logstash-plugins](https://github.com/logstash-plugins)

### logstash inputs 配置
- stdin 
- file
- tcp／udp
- rsyslog
- redis
- kafka 
- beats

```
input {
	stdin {
	} 
}
outpu {
	stdout {
	} 
}

```

#### stdin 
```
stdin {
	add_field => { "a" => "b" }
	codec => "json"
	tags => "["a","b"]"
	type => "my_type"

} 
```
#### file 

- close_older number        	  No
- delimiter string         	  No
- discover_interval number      No 
- exclude array             	  No
- ignore_older number           No
- max_open_files number         No
- path array                    Yes
- sincedb_path string           No
- sincedb_write_interval number No
- start_position string, one of ["beginning", "end"] No
- stat_interval number          No

```
input {
	file {
		path => ["/var/log/nginx/access.log"] 
		type => "nginx-log"  
		start_position => 'beginning'
	} 
} 
output {
	stdout {} 
}


```

#### tcp/udp
```
input {
	tcp {
		port => 9090 
		mode => "server"
		ssl_enable => false 
		
	}
}
output {
	stdout {} 
	
} 
nc 127.0.0.1:9090 < data

input {
	udp {
		host => "127.0.0.1" 
		port => 5050		
	}	
} 
output {
	stdout {} 
	
} 

```
```
#python udp客户端
import socket 
port = 5050
host = "127.0.0.1"
file_input = raw_input("\033[32;1mPlease input: \033[0m")
s = socket.socket(socket.AF_INET,socket_SOCK_DGRAM) 
s.sendto(file_input,(host,port))

```
#### rsyslog 
```
input {
	syslog {
		host => "127.0.0.1" 
		type => "syslog" 
		port => 518 
		
	} 

} 
output {
	stdout { } 

} 
###
vim /etc/rsyslog.conf 
*.* @@127.0.0.1:518
### 
logger 命令模拟发送日志
```
#### 编码 

```
# plain
input {
	stdin {
		codec => 'plain'
	} 
} 
output {
	stdout { }  
} 
```  
```
# json
input {
	stdin {} 

} 
output {
	stdout {
		codec => "json" 
	}

} 
```
```
#json_lines
input {
	tcp {
		port => 12345
		host => '127.0.0.1'
		codec => json_lines 
	
	}
}
output {
	stdout { } 
}
```
```
#rubydebug 
input {
	stdin {
		codec => json 
	}
} 
output {
	stdout {
		codec => rubydebug 
	}
} 
```
#### multiline 

```
input {
	stdin {
		codec => multiline {
			charset => ""     #字符编码
			max_bytes => 		#最大字节数
			max_lines =>      #最大行数，默认500
			multiline_tag =>  #设置一个事件标签，默认multiline
			pattern =>   	    #string匹配规则
			patterns_dir =>    #array多个匹配规则
			negate => false   #设置正向匹配还是反向匹配
			what   => next    #匹配的内容后，后面多行的日志是向前靠拢还是向后靠拢，previous,next
		}
	
	
	}
}
```
```
input {
	stdin {
		codec = multiline {
			pattern => "^\["
			negate => true
			what => previous
		
		}
	
	}
} 
output {
	stdout {
		codec => rubydebug 
	
	} 
} 
```
### logstash filter 配置

- json file
- grok file
- kv file

#### grok filter 




#### kv filter 

### logstash output 配置 

- file输出
- tcp／udp方式输出
- elasticsearch
- redis
- kafka
- hdfs
- email



#### output file

```
output {
	file {
		path => "/root/access_result"
		#message_format => "%{ip}" 
		#path => "/root/access_%{+YYYY.MM.DD}_%{host}.txt"  
        #gzip => true 
	
	}
	stdout {
		codec => rebydebug 
		
	}
}
```

``` 
output {
	tcp {
		codec => json_lines 
		host => "127.0.0.1"
		port => "4050"
		mode => "server"
	}

}

```
```
output {
		udp {
			host => "127.0.0.1"
			port => 4050
		}
} 
```

```
output {
	elasticsearch {
		host => "127.0.0.1" 
		protocol => "http" 
		index => "test_output-%{type}-%{+YYYY.MM.dd}"
		document_type => "nginx" 
		workers => 5

	}
}
```
