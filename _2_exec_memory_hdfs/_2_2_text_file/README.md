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
total 4
-rw-rw-r-- 1 cloudera cloudera 611 May  6 06:57 exec_memory_hdfs_textfile.conf
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
  -f /home/cloudera/flume_demo/exec_memory_hdfs.conf \
  -Dflume.root.logger=INFO,console

~~~

### Step-6: Verify logs under HDFS

* **Verify files (generated by flume agent) under HDFS**

~~~
[cloudera@quickstart ~]$ hadoop fs -ls /user/cloudera/flume_demo

~~~

* **View file data under HDFS**

~~~
[cloudera@quickstart ~]$ hadoop fs -cat /user/cloudera/flume_demo/FlumeData.1525622241522

~~~