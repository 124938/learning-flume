## Problem Statement 
* Read data from web log file and write it to HDFS location (in text file format & custom hdfs configuration) using memory channel

## Solution
 
### Step-0: Login to Quick Start VM OR Gateway Node of hadoop cluster

~~~
asus@asus-GL553VD:~$ ssh cloudera@192.168.211.142
cloudera@192.168.211.142's password: 
Last login: Sun May  6 08:35:07 2018 from 192.168.211.1

[cloudera@quickstart ~]$ 
~~~

### Step-1: Make sure to have web log generation script

~~~
[cloudera@quickstart flume_demo]$ ls -ltr -R /opt/gen_logs/
/opt/gen_logs/:
total 24
-rwxr-xr-x 1 cloudera cloudera   51 Aug  1  2014 tail_logs.sh
drwxr-xr-x 2 cloudera cloudera 4096 Sep 25  2014 logs
drwxr-xr-x 2 cloudera cloudera 4096 Sep 25  2014 data
-rwxr-xr-x 1 cloudera cloudera   76 Oct  8  2014 start_logs.sh
-rwxr-xr-x 1 cloudera cloudera  131 May 14  2015 stop_logs.sh
drwxr-xr-x 2 cloudera cloudera 4096 Jul 19  2017 lib

/opt/gen_logs/logs:
total 512
-rw-r--r-- 1 cloudera cloudera 518035 May  7 06:52 access.log

/opt/gen_logs/data:
total 64
-rw-r--r-- 1 cloudera cloudera 55172 Sep 25  2014 products.json
-rw-r--r-- 1 cloudera cloudera   280 Sep 25  2014 departments.json
-rw-r--r-- 1 cloudera cloudera  2433 Sep 25  2014 categories.json

/opt/gen_logs/lib:
total 8
-rwxrwxr-x 1 cloudera cloudera 6179 Oct 21  2014 genhttplogs.py
~~~

~~~
[cloudera@quickstart logs]$ tail -F /opt/gen_logs/logs/access.log 

194.230.153.123 - - [07/May/2018:06:53:29 -0800] "GET /product/1066 HTTP/1.1" 200 722 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
169.170.80.142 - - [07/May/2018:06:53:30 -0800] "GET /departments HTTP/1.1" 200 390 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:30.0) Gecko/20100101 Firefox/30.0"
146.233.193.82 - - [07/May/2018:06:53:31 -0800] "GET /categories/lacrosse/products HTTP/1.1" 200 2186 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
175.229.139.10 - - [07/May/2018:06:53:32 -0800] "GET /login HTTP/1.1" 200 210 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36"
62.135.121.193 - - [07/May/2018:06:53:33 -0800] "GET /department/apparel/products HTTP/1.1" 200 796 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
67.148.180.164 - - [07/May/2018:06:53:34 -0800] "GET /departments HTTP/1.1" 200 1881 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:30.0) Gecko/20100101 Firefox/30.0"
1.0.2.155 - - [07/May/2018:06:53:35 -0800] "GET /departments HTTP/1.1" 200 1560 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36"
144.41.159.79 - - [07/May/2018:06:53:36 -0800] "GET /login HTTP/1.1" 503 404 "-" "Mozilla/5.0 (Windows NT 6.1; rv:30.0) Gecko/20100101 Firefox/30.0"
132.103.234.79 - - [07/May/2018:06:53:37 -0800] "GET /categories/golf%20gloves/products HTTP/1.1" 200 655 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.76.4 (KHTML, like Gecko) Version/7.0.4 Safari/537.76.4"
215.209.179.217 - - [07/May/2018:06:53:38 -0800] "GET /department/golf/categories HTTP/1.1" 200 2151 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
~~~

### Step-2: Develope flume specific configuration file

* **Create directory (if not exists)**

~~~
[cloudera@quickstart ~]$ mkdir flume_demo
~~~

* **Change directory**

~~~
[cloudera@quickstart ~]$ cd flume_demo/
~~~

* **Create flume configuration file**

~~~
[cloudera@quickstart flume_demo]$ vi exec_memory_hdfs_textfile.conf
~~~

~~~
# Name the components on this agent
a3.sources = exec_source
a3.sinks = hdfs_sink
a3.channels = mem_channel

# Describe/configure the source
a3.sources.exec_source.type = exec
a3.sources.exec_source.command = tail -F /opt/gen_logs/logs/access.log

# Describe the sink
a3.sinks.hdfs_sink.type = hdfs
a3.sinks.hdfs_sink.hdfs.path = hdfs://quickstart.cloudera:8020/user/cloudera/flume_demo
a3.sinks.hdfs_sink.hdfs.filePrefix = flume-demo
a3.sinks.hdfs_sink.hdfs.fileSuffix = .txt
a3.sinks.hdfs_sink.hdfs.rollInterval = 120
a3.sinks.hdfs_sink.hdfs.rollSize = 1048576
a3.sinks.hdfs_sink.hdfs.rollCount = 100
a3.sinks.hdfs_sink.hdfs.fileType = DataStream

# Use a channel which buffers events in memory
a3.channels.mem_channel.type = memory
a3.channels.mem_channel.capacity = 1000
a3.channels.mem_channel.transactionCapacity = 100

# Bind the source and sink to the channel
a3.sources.exec_source.channels = mem_channel
a3.sinks.hdfs_sink.channel = mem_channel
~~~

~~~
[cloudera@quickstart flume_demo]$ ls -ltr
total 12
-rw-rw-r-- 1 cloudera cloudera 617 May  6 07:06 netcat_memory_logger.conf
-rw-rw-r-- 1 cloudera cloudera 694 May  6 08:54 exec_memory_hdfs_sequencefile.conf
-rw-rw-r-- 1 cloudera cloudera 956 May  7 07:28 exec_memory_hdfs_textfile.conf
~~~

### Step-3: Create HDFS directory

~~~
[cloudera@quickstart ~]$ hadoop fs -rm -R /user/cloudera/flume_demo
Deleted /user/cloudera/flume_demo

[cloudera@quickstart ~]$ hadoop fs -mkdir /user/cloudera/flume_demo
~~~

### Step-4: Start log generator script

~~~
[cloudera@quickstart ~]$ cd /opt/gen_logs

[cloudera@quickstart ~]$ ./start_logs.sh
~~~

### Step-5: Start flume agent

~~~
[cloudera@quickstart ~]$ flume-ng agent \
  -n a3 \
  -f /home/cloudera/flume_demo/exec_memory_hdfs_textfile.conf \
  -Dflume.root.logger=INFO,console

Warning: No configuration directory set! Use --conf <dir> to override.
Info: Including Hadoop libraries found via (/usr/bin/hadoop) for HDFS access
Info: Including HBASE libraries found via (/usr/bin/hbase) for HBASE access
Info: Including Hive libraries found via () for Hive access
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/lib/flume-ng/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/lib/zookeeper/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
18/05/07 07:31:47 INFO node.PollingPropertiesFileConfigurationProvider: Configuration provider starting
18/05/07 07:31:47 INFO node.PollingPropertiesFileConfigurationProvider: Reloading configuration file:/home/cloudera/flume_demo/exec_memory_hdfs_textfile.conf
18/05/07 07:31:47 INFO conf.FlumeConfiguration: Processing:hdfs_sink
18/05/07 07:31:47 INFO conf.FlumeConfiguration: Processing:hdfs_sink
18/05/07 07:31:47 INFO conf.FlumeConfiguration: Processing:hdfs_sink
18/05/07 07:31:47 INFO conf.FlumeConfiguration: Processing:hdfs_sink
18/05/07 07:31:47 INFO conf.FlumeConfiguration: Processing:hdfs_sink
18/05/07 07:31:47 INFO conf.FlumeConfiguration: Processing:hdfs_sink
18/05/07 07:31:47 INFO conf.FlumeConfiguration: Processing:hdfs_sink
18/05/07 07:31:47 INFO conf.FlumeConfiguration: Processing:hdfs_sink
18/05/07 07:31:47 INFO conf.FlumeConfiguration: Added sinks: hdfs_sink Agent: a3
18/05/07 07:31:47 INFO conf.FlumeConfiguration: Processing:hdfs_sink
18/05/07 07:31:48 INFO conf.FlumeConfiguration: Post-validation flume configuration contains configuration for agents: [a3]
18/05/07 07:31:48 INFO node.AbstractConfigurationProvider: Creating channels
18/05/07 07:31:48 INFO channel.DefaultChannelFactory: Creating instance of channel mem_channel type memory
18/05/07 07:31:48 INFO node.AbstractConfigurationProvider: Created channel mem_channel
18/05/07 07:31:48 INFO source.DefaultSourceFactory: Creating instance of source exec_source, type exec
18/05/07 07:31:48 INFO sink.DefaultSinkFactory: Creating instance of sink: hdfs_sink, type: hdfs
18/05/07 07:31:48 INFO node.AbstractConfigurationProvider: Channel mem_channel connected to [exec_source, hdfs_sink]
18/05/07 07:31:48 INFO node.Application: Starting new configuration:{ sourceRunners:{exec_source=EventDrivenSourceRunner: { source:org.apache.flume.source.ExecSource{name:exec_source,state:IDLE} }} sinkRunners:{hdfs_sink=SinkRunner: { policy:org.apache.flume.sink.DefaultSinkProcessor@42a4441e counterGroup:{ name:null counters:{} } }} channels:{mem_channel=org.apache.flume.channel.MemoryChannel{name: mem_channel}} }
18/05/07 07:31:48 INFO node.Application: Starting Channel mem_channel
18/05/07 07:31:48 INFO instrumentation.MonitoredCounterGroup: Monitored counter group for type: CHANNEL, name: mem_channel: Successfully registered new MBean.
18/05/07 07:31:48 INFO instrumentation.MonitoredCounterGroup: Component type: CHANNEL, name: mem_channel started
18/05/07 07:31:48 INFO node.Application: Starting Sink hdfs_sink
18/05/07 07:31:48 INFO node.Application: Starting Source exec_source
18/05/07 07:31:48 INFO source.ExecSource: Exec source starting with command: tail -F /opt/gen_logs/logs/access.log
18/05/07 07:31:48 INFO instrumentation.MonitoredCounterGroup: Monitored counter group for type: SOURCE, name: exec_source: Successfully registered new MBean.
18/05/07 07:31:48 INFO instrumentation.MonitoredCounterGroup: Component type: SOURCE, name: exec_source started
18/05/07 07:31:48 INFO instrumentation.MonitoredCounterGroup: Monitored counter group for type: SINK, name: hdfs_sink: Successfully registered new MBean.
18/05/07 07:31:48 INFO instrumentation.MonitoredCounterGroup: Component type: SINK, name: hdfs_sink started
18/05/07 07:31:52 INFO hdfs.HDFSDataStream: Serializer = TEXT, UseRawLocalFileSystem = false
18/05/07 07:31:52 INFO hdfs.BucketWriter: Creating hdfs://quickstart.cloudera:8020/user/cloudera/flume_demo/flume-demo.1525703512106.txt.tmp
~~~

### Step-6: Verify logs under HDFS

* **Verify files (generated by flume agent) under HDFS**

~~~
[cloudera@quickstart ~]$ hadoop fs -ls /user/cloudera/flume_demo
Found 2 items
-rw-r--r--   1 cloudera cloudera      19994 2018-05-07 07:33 /user/cloudera/flume_demo/flume-demo.1525703512106.txt
-rw-r--r--   1 cloudera cloudera          0 2018-05-07 07:33 /user/cloudera/flume_demo/flume-demo.1525703512107.txt.tmp

[cloudera@quickstart ~]$ hadoop fs -cat /user/cloudera/flume_demo/flume-demo.1525703512106.txt | wc -l
100
~~~

* **View file data under HDFS**

~~~
[cloudera@quickstart ~]$ hadoop fs -cat /user/cloudera/flume_demo/flume-demo.1525703512106.txt

28.85.153.126 - - [07/May/2018:07:31:38 -0800] "GET /departments HTTP/1.1" 200 540 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.76.4 (KHTML, like Gecko) Version/7.0.4 Safari/537.76.4"
156.95.126.144 - - [07/May/2018:07:31:39 -0800] "GET /departments HTTP/1.1" 200 1708 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
199.232.172.24 - - [07/May/2018:07:31:40 -0800] "GET /department/team%20sports/categories HTTP/1.1" 200 2226 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36"
206.195.56.29 - - [07/May/2018:07:31:41 -0800] "GET /department/apparel/categories HTTP/1.1" 200 233 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
112.147.227.38 - - [07/May/2018:07:31:42 -0800] "GET /product/446 HTTP/1.1" 200 1446 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
183.24.51.251 - - [07/May/2018:07:31:43 -0800] "GET /department/outdoors/categories HTTP/1.1" 200 1115 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko"
177.14.197.211 - - [07/May/2018:07:31:44 -0800] "GET /categories/golf%20shoes/products HTTP/1.1" 200 1830 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
175.180.170.172 - - [07/May/2018:07:31:45 -0800] "GET /departments HTTP/1.1" 200 1779 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
176.168.123.168 - - [07/May/2018:07:31:46 -0800] "GET /departments HTTP/1.1" 200 1012 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
6.181.66.191 - - [07/May/2018:07:31:47 -0800] "GET /department/team%20sports/categories HTTP/1.1" 200 1135 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0"
88.200.25.184 - - [07/May/2018:07:31:48 -0800] "GET /checkout HTTP/1.1" 200 1195 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:30.0) Gecko/20100101 Firefox/30.0"
17.133.95.193 - - [07/May/2018:07:31:49 -0800] "GET /department/golf/categories HTTP/1.1" 200 360 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
125.200.203.96 - - [07/May/2018:07:31:50 -0800] "GET /department/outdoors/categories HTTP/1.1" 200 1665 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
87.87.182.71 - - [07/May/2018:07:31:51 -0800] "GET /department/golf/categories HTTP/1.1" 200 1624 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
30.20.52.192 - - [07/May/2018:07:31:52 -0800] "GET /checkout HTTP/1.1" 200 1475 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
164.132.48.139 - - [07/May/2018:07:31:53 -0800] "GET /departments HTTP/1.1" 200 1512 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
183.24.51.251 - - [07/May/2018:07:31:54 -0800] "GET /department/apparel/products HTTP/1.1" 200 721 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.76.4 (KHTML, like Gecko) Version/7.0.4 Safari/537.76.4"
122.175.178.81 - - [07/May/2018:07:31:55 -0800] "GET /login HTTP/1.1" 200 373 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.76.4 (KHTML, like Gecko) Version/7.0.4 Safari/537.76.4"
34.111.70.231 - - [07/May/2018:07:31:56 -0800] "GET /product/345 HTTP/1.1" 200 1610 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
23.106.140.21 - - [07/May/2018:07:31:57 -0800] "GET /department/outdoors/categories HTTP/1.1" 200 2234 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
112.167.244.136 - - [07/May/2018:07:31:58 -0800] "GET /logout HTTP/1.1" 200 1736 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
16.87.74.185 - - [07/May/2018:07:31:59 -0800] "GET /department/footwear/categories HTTP/1.1" 200 1294 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
140.56.183.198 - - [07/May/2018:07:32:00 -0800] "GET /add_to_cart/708 HTTP/1.1" 200 492 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36"
141.153.197.58 - - [07/May/2018:07:32:01 -0800] "GET /department/apparel/categories HTTP/1.1" 200 202 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
28.85.153.126 - - [07/May/2018:07:32:02 -0800] "GET /support HTTP/1.1" 200 311 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
112.25.114.171 - - [07/May/2018:07:32:03 -0800] "GET /departments HTTP/1.1" 200 919 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0"
13.139.0.219 - - [07/May/2018:07:32:04 -0800] "GET /product/728 HTTP/1.1" 200 2011 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.77.4 (KHTML, like Gecko) Version/7.0.5 Safari/537.77.4"
38.45.133.199 - - [07/May/2018:07:32:05 -0800] "GET /department/fan%20shop/products HTTP/1.1" 200 683 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.77.4 (KHTML, like Gecko) Version/7.0.5 Safari/537.77.4"
125.200.203.96 - - [07/May/2018:07:32:06 -0800] "GET /departments HTTP/1.1" 200 497 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36"
68.117.17.178 - - [07/May/2018:07:32:07 -0800] "GET /departments HTTP/1.1" 200 1233 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
108.246.168.145 - - [07/May/2018:07:32:08 -0800] "GET /product/861 HTTP/1.1" 200 1274 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
112.25.114.171 - - [07/May/2018:07:32:09 -0800] "GET /add_to_cart/704 HTTP/1.1" 200 1169 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0"
35.188.98.238 - - [07/May/2018:07:32:10 -0800] "GET /department/golf/categories HTTP/1.1" 200 1530 "-" "Mozilla/5.0 (Windows NT 6.1; rv:30.0) Gecko/20100101 Firefox/30.0"
59.77.144.119 - - [07/May/2018:07:32:11 -0800] "GET /login HTTP/1.1" 200 1131 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0"
6.181.66.191 - - [07/May/2018:07:32:12 -0800] "GET /departments HTTP/1.1" 200 1521 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
9.68.77.112 - - [07/May/2018:07:32:13 -0800] "GET /departments HTTP/1.1" 200 1896 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
102.151.235.90 - - [07/May/2018:07:32:14 -0800] "GET /department/fitness/products HTTP/1.1" 200 1987 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.77.4 (KHTML, like Gecko) Version/7.0.5 Safari/537.77.4"
72.171.38.88 - - [07/May/2018:07:32:15 -0800] "GET /departments HTTP/1.1" 404 1973 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
199.232.172.24 - - [07/May/2018:07:32:16 -0800] "GET /product/191 HTTP/1.1" 200 1943 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36"
70.61.79.167 - - [07/May/2018:07:32:17 -0800] "GET /departments HTTP/1.1" 200 1843 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36"
211.5.144.254 - - [07/May/2018:07:32:18 -0800] "GET /departments HTTP/1.1" 200 1738 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
114.203.13.253 - - [07/May/2018:07:32:19 -0800] "GET /departments HTTP/1.1" 200 1336 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
59.77.144.119 - - [07/May/2018:07:32:20 -0800] "GET /product/158 HTTP/1.1" 200 1418 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.76.4 (KHTML, like Gecko) Version/7.0.4 Safari/537.76.4"
112.147.227.38 - - [07/May/2018:07:32:21 -0800] "GET /checkout HTTP/1.1" 200 262 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36"
48.19.75.156 - - [07/May/2018:07:32:22 -0800] "GET /add_to_cart/491 HTTP/1.1" 200 289 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.77.4 (KHTML, like Gecko) Version/7.0.5 Safari/537.77.4"
70.61.79.167 - - [07/May/2018:07:32:23 -0800] "GET /departments HTTP/1.1" 200 1438 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.77.4 (KHTML, like Gecko) Version/7.0.5 Safari/537.77.4"
164.132.48.139 - - [07/May/2018:07:32:24 -0800] "GET /department/apparel/categories HTTP/1.1" 200 467 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0"
95.37.23.13 - - [07/May/2018:07:32:25 -0800] "GET /department/apparel/categories HTTP/1.1" 200 1181 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
98.24.106.187 - - [07/May/2018:07:32:26 -0800] "GET /department/footwear/categories HTTP/1.1" 200 1894 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.76.4 (KHTML, like Gecko) Version/7.0.4 Safari/537.76.4"
85.176.49.41 - - [07/May/2018:07:32:27 -0800] "GET /departments HTTP/1.1" 200 647 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0"
79.70.193.184 - - [07/May/2018:07:32:28 -0800] "GET /departments HTTP/1.1" 200 695 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko"
121.52.208.71 - - [07/May/2018:07:32:29 -0800] "GET /department/outdoors/products HTTP/1.1" 200 1869 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
176.168.123.168 - - [07/May/2018:07:32:30 -0800] "GET /department/outdoors/categories HTTP/1.1" 200 1177 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:30.0) Gecko/20100101 Firefox/30.0"
7.161.147.102 - - [07/May/2018:07:32:31 -0800] "GET /categories/boys%27%20apparel/products HTTP/1.1" 200 1589 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.76.4 (KHTML, like Gecko) Version/7.0.4 Safari/537.76.4"
48.19.75.156 - - [07/May/2018:07:32:32 -0800] "GET /support HTTP/1.1" 200 1724 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
223.65.125.196 - - [07/May/2018:07:32:33 -0800] "GET /departments HTTP/1.1" 200 1787 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0"
156.95.126.144 - - [07/May/2018:07:32:34 -0800] "GET /add_to_cart/461 HTTP/1.1" 200 365 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko"
100.99.215.206 - - [07/May/2018:07:32:35 -0800] "GET /departments HTTP/1.1" 200 583 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
10.61.246.91 - - [07/May/2018:07:32:36 -0800] "GET /departments HTTP/1.1" 200 734 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0"
8.171.128.70 - - [07/May/2018:07:32:37 -0800] "GET /categories/golf%20balls/products HTTP/1.1" 200 761 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
175.180.170.172 - - [07/May/2018:07:32:38 -0800] "GET /department/team%20sports/categories HTTP/1.1" 200 1886 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
16.87.74.185 - - [07/May/2018:07:32:39 -0800] "GET /login HTTP/1.1" 200 898 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
177.14.197.211 - - [07/May/2018:07:32:40 -0800] "GET /departments HTTP/1.1" 503 619 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
155.197.239.218 - - [07/May/2018:07:32:41 -0800] "GET /departments HTTP/1.1" 200 1269 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
211.133.101.216 - - [07/May/2018:07:32:42 -0800] "GET /departments HTTP/1.1" 200 1628 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
178.220.96.133 - - [07/May/2018:07:32:43 -0800] "GET /departments HTTP/1.1" 200 225 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.77.4 (KHTML, like Gecko) Version/7.0.5 Safari/537.77.4"
206.195.56.29 - - [07/May/2018:07:32:44 -0800] "GET /departments HTTP/1.1" 200 584 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
17.133.95.193 - - [07/May/2018:07:32:45 -0800] "GET /departments HTTP/1.1" 200 1055 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
211.133.101.216 - - [07/May/2018:07:32:46 -0800] "GET /categories/bike%20%26%20skate%20shop/products HTTP/1.1" 200 1964 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
17.133.95.193 - - [07/May/2018:07:32:47 -0800] "GET /departments HTTP/1.1" 200 1904 "-" "Mozilla/5.0 (Windows NT 6.1; rv:30.0) Gecko/20100101 Firefox/30.0"
211.133.101.216 - - [07/May/2018:07:32:48 -0800] "GET /departments HTTP/1.1" 200 690 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
68.117.17.178 - - [07/May/2018:07:32:49 -0800] "GET /departments HTTP/1.1" 200 2009 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.77.4 (KHTML, like Gecko) Version/7.0.5 Safari/537.77.4"
112.189.135.122 - - [07/May/2018:07:32:50 -0800] "GET /departments HTTP/1.1" 200 2062 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0"
46.96.49.99 - - [07/May/2018:07:32:51 -0800] "GET /departments HTTP/1.1" 200 245 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
89.227.56.143 - - [07/May/2018:07:32:52 -0800] "GET /departments HTTP/1.1" 200 1613 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
155.40.83.43 - - [07/May/2018:07:32:53 -0800] "GET /categories/electronics/products HTTP/1.1" 200 1854 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
70.61.79.167 - - [07/May/2018:07:32:54 -0800] "GET /departments HTTP/1.1" 200 1974 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
25.242.215.62 - - [07/May/2018:07:32:55 -0800] "GET /departments HTTP/1.1" 200 1464 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
19.67.122.192 - - [07/May/2018:07:32:56 -0800] "GET /product/858 HTTP/1.1" 200 442 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0"
176.168.123.168 - - [07/May/2018:07:32:57 -0800] "GET /departments HTTP/1.1" 200 2011 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.76.4 (KHTML, like Gecko) Version/7.0.4 Safari/537.76.4"
10.61.246.91 - - [07/May/2018:07:32:58 -0800] "GET /department/apparel/categories HTTP/1.1" 200 1682 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
223.65.125.196 - - [07/May/2018:07:32:59 -0800] "GET /department/footwear/categories HTTP/1.1" 200 260 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
10.61.246.91 - - [07/May/2018:07:33:00 -0800] "GET /departments HTTP/1.1" 200 1464 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
161.198.167.59 - - [07/May/2018:07:33:01 -0800] "GET /product/412 HTTP/1.1" 200 965 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.76.4 (KHTML, like Gecko) Version/7.0.4 Safari/537.76.4"
184.125.187.218 - - [07/May/2018:07:33:02 -0800] "GET /categories/international%20soccer/products HTTP/1.1" 200 2017 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
10.61.246.91 - - [07/May/2018:07:33:03 -0800] "GET /departments HTTP/1.1" 503 1964 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
216.49.84.123 - - [07/May/2018:07:33:04 -0800] "GET /departments HTTP/1.1" 200 1751 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.77.4 (KHTML, like Gecko) Version/7.0.5 Safari/537.77.4"
178.220.96.133 - - [07/May/2018:07:33:05 -0800] "GET /checkout HTTP/1.1" 200 1496 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:30.0) Gecko/20100101 Firefox/30.0"
78.139.46.37 - - [07/May/2018:07:33:06 -0800] "GET /login HTTP/1.1" 200 468 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko"
112.189.135.122 - - [07/May/2018:07:33:07 -0800] "GET /department/fan%20shop/categories HTTP/1.1" 200 605 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko"
68.117.17.178 - - [07/May/2018:07:33:08 -0800] "GET /departments HTTP/1.1" 200 568 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36"
199.232.172.24 - - [07/May/2018:07:33:09 -0800] "GET /department/golf/products HTTP/1.1" 200 1322 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
108.246.168.145 - - [07/May/2018:07:33:10 -0800] "GET /departments HTTP/1.1" 200 1624 "-" "Mozilla/5.0 (Windows NT 6.1; rv:30.0) Gecko/20100101 Firefox/30.0"
205.92.216.235 - - [07/May/2018:07:33:11 -0800] "GET /product/223 HTTP/1.1" 200 1567 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0"
16.87.74.185 - - [07/May/2018:07:33:12 -0800] "GET /departments HTTP/1.1" 200 1109 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
10.61.246.91 - - [07/May/2018:07:33:13 -0800] "GET /login HTTP/1.1" 200 2111 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
112.189.135.122 - - [07/May/2018:07:33:14 -0800] "GET /departments HTTP/1.1" 200 1025 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
48.19.75.156 - - [07/May/2018:07:33:15 -0800] "GET /departments HTTP/1.1" 200 1834 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.76.4 (KHTML, like Gecko) Version/7.0.4 Safari/537.76.4"
140.56.183.198 - - [07/May/2018:07:33:16 -0800] "GET /department/outdoors/products HTTP/1.1" 200 869 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko"
44.187.41.56 - - [07/May/2018:07:33:17 -0800] "GET /department/fitness/products HTTP/1.1" 200 569 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
~~~