## Problem Statement 
* Read data from web log file and write it to HDFS location (in sequence file format) using memory channel

## Solution
 
### Step-0: Login to Quick Start VM OR Gateway Node of hadoop cluster

~~~
asus@asus-GL553VD:~$ ssh cloudera@192.168.211.142
cloudera@192.168.211.142's password: 
Last login: Sun May  6 04:51:45 2018 from 192.168.211.1

[cloudera@quickstart ~]$ 
~~~

### Step-1: Make sure to have web log generation script

~~~
[cloudera@quickstart ~]$ ls -ltr -R /opt/gen_logs/
/opt/gen_logs/:
total 24
-rwxr-xr-x 1 cloudera cloudera   51 Aug  1  2014 tail_logs.sh
drwxr-xr-x 2 cloudera cloudera 4096 Sep 25  2014 logs
drwxr-xr-x 2 cloudera cloudera 4096 Sep 25  2014 data
-rwxr-xr-x 1 cloudera cloudera   76 Oct  8  2014 start_logs.sh
-rwxr-xr-x 1 cloudera cloudera  131 May 14  2015 stop_logs.sh
drwxr-xr-x 2 cloudera cloudera 4096 Jul 19  2017 lib

/opt/gen_logs/logs:
total 4
-rw-r--r-- 1 cloudera cloudera 3093 May  6 08:35 access.log

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

223.132.60.21 - - [06/May/2018:08:35:40 -0800] "GET /departments HTTP/1.1" 200 760 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
132.151.66.53 - - [06/May/2018:08:35:41 -0800] "GET /login HTTP/1.1" 200 921 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36"
146.131.54.84 - - [06/May/2018:08:35:42 -0800] "GET /department/team%20sports/products HTTP/1.1" 200 382 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
193.152.73.182 - - [06/May/2018:08:35:43 -0800] "GET /departments HTTP/1.1" 200 1686 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
119.87.97.175 - - [06/May/2018:08:35:44 -0800] "GET /departments HTTP/1.1" 200 1598 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
8.233.230.103 - - [06/May/2018:08:35:45 -0800] "GET /department/golf/categories HTTP/1.1" 200 859 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0"
118.126.50.149 - - [06/May/2018:08:35:46 -0800] "GET /department/team%20sports/categories HTTP/1.1" 200 537 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
133.150.225.15 - - [06/May/2018:08:35:47 -0800] "GET /departments HTTP/1.1" 200 2003 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
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
[cloudera@quickstart flume_demo]$ vi exec_memory_hdfs_sequencefile.conf
~~~

~~~
# Name the components on this agent
a2.sources = exec_source
a2.sinks = hdfs_sink
a2.channels = mem_channel

# Describe/configure the source
a2.sources.exec_source.type = exec
a2.sources.exec_source.command = tail -F /opt/gen_logs/logs/access.log

# Describe the sink
a2.sinks.hdfs_sink.type = hdfs
a2.sinks.hdfs_sink.hdfs.path = hdfs://quickstart.cloudera:8020/user/cloudera/flume_demo

# Use a channel which buffers events in memory
a2.channels.mem_channel.type = memory
a2.channels.mem_channel.capacity = 1000
a2.channels.mem_channel.transactionCapacity = 100

# Bind the source and sink to the channel
a2.sources.exec_source.channels = mem_channel
a2.sinks.hdfs_sink.channel = mem_channel
~~~

~~~
[cloudera@quickstart flume_demo]$ ls -ltr
total 8
-rw-rw-r-- 1 cloudera cloudera 617 May  6 07:06 netcat_memory_logger.conf
-rw-rw-r-- 1 cloudera cloudera 694 May  6 08:54 exec_memory_hdfs_sequencefile.conf
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
  -n a2 \
  -f /home/cloudera/flume_demo/exec_memory_hdfs_sequencefile.conf \
  -Dflume.root.logger=INFO,console

Warning: No configuration directory set! Use --conf <dir> to override.
Info: Including Hadoop libraries found via (/usr/bin/hadoop) for HDFS access
Info: Including HBASE libraries found via (/usr/bin/hbase) for HBASE access
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/lib/flume-ng/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/lib/zookeeper/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
18/05/06 08:57:17 INFO node.PollingPropertiesFileConfigurationProvider: Configuration provider starting
18/05/06 08:57:17 INFO node.PollingPropertiesFileConfigurationProvider: Reloading configuration file:/home/cloudera/flume_demo/exec_memory_hdfs_sequencefile.conf
18/05/06 08:57:17 INFO conf.FlumeConfiguration: Processing:hdfs_sink
18/05/06 08:57:17 INFO conf.FlumeConfiguration: Added sinks: hdfs_sink Agent: a2
18/05/06 08:57:17 INFO conf.FlumeConfiguration: Processing:hdfs_sink
18/05/06 08:57:17 INFO conf.FlumeConfiguration: Processing:hdfs_sink
18/05/06 08:57:17 INFO conf.FlumeConfiguration: Post-validation flume configuration contains configuration for agents: [a2]
18/05/06 08:57:17 INFO node.AbstractConfigurationProvider: Creating channels
18/05/06 08:57:17 INFO channel.DefaultChannelFactory: Creating instance of channel mem_channel type memory
18/05/06 08:57:17 INFO node.AbstractConfigurationProvider: Created channel mem_channel
18/05/06 08:57:17 INFO source.DefaultSourceFactory: Creating instance of source exec_source, type exec
18/05/06 08:57:17 INFO sink.DefaultSinkFactory: Creating instance of sink: hdfs_sink, type: hdfs
18/05/06 08:57:17 INFO node.AbstractConfigurationProvider: Channel mem_channel connected to [exec_source, hdfs_sink]
18/05/06 08:57:17 INFO node.Application: Starting new configuration:{ sourceRunners:{exec_source=EventDrivenSourceRunner: { source:org.apache.flume.source.ExecSource{name:exec_source,state:IDLE} }} sinkRunners:{hdfs_sink=SinkRunner: { policy:org.apache.flume.sink.DefaultSinkProcessor@42a4441e counterGroup:{ name:null counters:{} } }} channels:{mem_channel=org.apache.flume.channel.MemoryChannel{name: mem_channel}} }
18/05/06 08:57:17 INFO node.Application: Starting Channel mem_channel
18/05/06 08:57:17 INFO instrumentation.MonitoredCounterGroup: Monitored counter group for type: CHANNEL, name: mem_channel: Successfully registered new MBean.
18/05/06 08:57:17 INFO instrumentation.MonitoredCounterGroup: Component type: CHANNEL, name: mem_channel started
18/05/06 08:57:17 INFO node.Application: Starting Sink hdfs_sink
18/05/06 08:57:17 INFO node.Application: Starting Source exec_source
18/05/06 08:57:17 INFO source.ExecSource: Exec source starting with command: tail -F /opt/gen_logs/logs/access.log
18/05/06 08:57:17 INFO instrumentation.MonitoredCounterGroup: Monitored counter group for type: SOURCE, name: exec_source: Successfully registered new MBean.
18/05/06 08:57:17 INFO instrumentation.MonitoredCounterGroup: Component type: SOURCE, name: exec_source started
18/05/06 08:57:17 INFO instrumentation.MonitoredCounterGroup: Monitored counter group for type: SINK, name: hdfs_sink: Successfully registered new MBean.
18/05/06 08:57:17 INFO instrumentation.MonitoredCounterGroup: Component type: SINK, name: hdfs_sink started
18/05/06 08:57:21 INFO hdfs.HDFSSequenceFile: writeFormat = Writable, UseRawLocalFileSystem = false
18/05/06 08:57:21 INFO hdfs.BucketWriter: Creating hdfs://quickstart.cloudera:8020/user/cloudera/flume_demo/FlumeData.1525622241522.tmp
18/05/06 08:57:23 INFO hdfs.BucketWriter: Closing hdfs://quickstart.cloudera:8020/user/cloudera/flume_demo/FlumeData.1525622241522.tmp
18/05/06 08:57:23 INFO hdfs.BucketWriter: Renaming hdfs://quickstart.cloudera:8020/user/cloudera/flume_demo/FlumeData.1525622241522.tmp to hdfs://quickstart.cloudera:8020/user/cloudera/flume_demo/FlumeData.1525622241522
18/05/06 08:57:24 INFO hdfs.BucketWriter: Creating hdfs://quickstart.cloudera:8020/user/cloudera/flume_demo/FlumeData.1525622241523.tmp
18/05/06 08:57:24 INFO hdfs.BucketWriter: Closing hdfs://quickstart.cloudera:8020/user/cloudera/flume_demo/FlumeData.1525622241523.tmp
~~~

### Step-6: Verify logs under HDFS

* **Verify files (generated by flume agent) under HDFS**

~~~
[cloudera@quickstart ~]$ hadoop fs -ls /user/cloudera/flume_demo
Found 7 items
-rw-r--r--   1 cloudera cloudera       1397 2018-05-06 08:57 /user/cloudera/flume_demo/FlumeData.1525622241522
-rw-r--r--   1 cloudera cloudera       1365 2018-05-06 08:57 /user/cloudera/flume_demo/FlumeData.1525622241523
-rw-r--r--   1 cloudera cloudera       1324 2018-05-06 08:57 /user/cloudera/flume_demo/FlumeData.1525622241524
-rw-r--r--   1 cloudera cloudera       1448 2018-05-06 08:57 /user/cloudera/flume_demo/FlumeData.1525622241525
-rw-r--r--   1 cloudera cloudera       1453 2018-05-06 08:57 /user/cloudera/flume_demo/FlumeData.1525622241526
-rw-r--r--   1 cloudera cloudera       1466 2018-05-06 08:57 /user/cloudera/flume_demo/FlumeData.1525622241527
-rw-r--r--   1 cloudera cloudera          0 2018-05-06 08:57 /user/cloudera/flume_demo/FlumeData.1525622241528.tmp
~~~

* **View file data under HDFS**

~~~
[cloudera@quickstart ~]$ hadoop fs -cat /user/cloudera/flume_demo/FlumeData.1525622241522
SEQ!org.apache.hadoop.io.LongWritable"org.apache.hadoop.io.BytesWritableX셴�3��������Ac6,��176.37.89.220 - - [06/May/2018:08:57:08 -0800] "GET /departments HTTP/1.1" 200 470 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"c6,��185.229.106.162 - - [06/May/2018:08:57:09 -0800] "GET /departments HTTP/1.1" 200 1849 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:30.0) Gecko/20100101 Firefox/30.0"c6,��164.205.139.35 - - [06/May/2018:08:57:10 -0800] "GET /add_to_cart/723 HTTP/1.1" 200 1011 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"c6,��20.216.96.62 - - [06/May/2018:08:57:11 -0800] "GET /categories/strength%20training/products HTTP/1.1" 200 1990 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0"c6,��30.68.150.12 - - [06/May/2018:08:57:12 -0800] "GET /departments HTTP/1.1" 200 1172 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"c6,��194.230.215.81 - - [06/May/2018:08:57:13 -0800] "GET /product/203 HTTP/1.1" 200 2232 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"����X셴�3��������A
~~~