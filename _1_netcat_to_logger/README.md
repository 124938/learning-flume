## Problem Statement : Read data from netcat source and log it to standard output of running flume agent
 
### Step-1: Login to Quick Start VM OR Gateway Node of hadoop cluster

~~~
asus@asus-GL553VD:~$ ssh cloudera@192.168.211.142
cloudera@192.168.211.142's password: 
Last login: Sun May  6 04:51:45 2018 from 192.168.211.1

[cloudera@quickstart ~]$ 
~~~

### Step-2: Create flume specific configuration file

* **Create directory**

~~~
[cloudera@quickstart ~]$ mkdir flume_demo
~~~

* **Change directory**

~~~
[cloudera@quickstart ~]$ cd flume_demo/
~~~

* **Create flume configuration file responsible to read data from netcat port and send data to STDOUT of flume**

~~~
[cloudera@quickstart flume_demo]$ vi netcat_to_logger.conf
~~~

~~~
# Name the components on this agent
a1.sources = nc_source
a1.sinks = stdout_sink
a1.channels = mem_channel

# Describe/configure the source
a1.sources.nc_source.type = netcat
a1.sources.nc_source.bind = 192.168.211.142
a1.sources.nc_source.port = 44444

# Describe the sink
a1.sinks.stdout_sink.type = logger

# Use a channel which buffers events in memory
a1.channels.mem_channel.type = memory
a1.channels.mem_channel.capacity = 1000
a1.channels.mem_channel.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.nc_source.channels = mem_channel
a1.sinks.stdout_sink.channel = mem_channel
~~~

~~~
[cloudera@quickstart flume_demo]$ ls -ltr
total 4
-rw-rw-r-- 1 cloudera cloudera 611 May  6 06:57 netcat_to_logger.conf
~~~

### Step-3: Start flume agent

* **Start flume agent**

~~~
[cloudera@quickstart ~]$ flume-ng agent \
  -n a1 \
  -f /home/cloudera/flume_demo/netcat_to_logger.conf

Warning: No configuration directory set! Use --conf <dir> to override.
Info: Including Hadoop libraries found via (/usr/bin/hadoop) for HDFS access
Info: Including HBASE libraries found via (/usr/bin/hbase) for HBASE access
Info: Including Hive libraries found via () for Hive access
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/lib/flume-ng/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/lib/zookeeper/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
18/05/06 07:00:13 INFO node.PollingPropertiesFileConfigurationProvider: Configuration provider starting
18/05/06 07:00:13 INFO node.PollingPropertiesFileConfigurationProvider: Reloading configuration file:/home/cloudera/flume_demo/netcat_to_logger.conf
18/05/06 07:00:13 INFO conf.FlumeConfiguration: Added sinks: stdout_sink Agent: a1
18/05/06 07:00:13 INFO conf.FlumeConfiguration: Processing:stdout_sink
18/05/06 07:00:13 INFO conf.FlumeConfiguration: Processing:stdout_sink
18/05/06 07:00:13 INFO conf.FlumeConfiguration: Post-validation flume configuration contains configuration for agents: [a1]
18/05/06 07:00:13 INFO node.AbstractConfigurationProvider: Creating channels
18/05/06 07:00:13 INFO channel.DefaultChannelFactory: Creating instance of channel mem_channel type memory
18/05/06 07:00:13 INFO node.AbstractConfigurationProvider: Created channel mem_channel
18/05/06 07:00:13 INFO source.DefaultSourceFactory: Creating instance of source nc_source, type netcat
18/05/06 07:00:13 INFO sink.DefaultSinkFactory: Creating instance of sink: stdout_sink, type: logger
18/05/06 07:00:13 INFO node.AbstractConfigurationProvider: Channel mem_channel connected to [nc_source, stdout_sink]
18/05/06 07:00:13 INFO node.Application: Starting new configuration:{ sourceRunners:{nc_source=EventDrivenSourceRunner: { source:org.apache.flume.source.NetcatSource{name:nc_source,state:IDLE} }} sinkRunners:{stdout_sink=SinkRunner: { policy:org.apache.flume.sink.DefaultSinkProcessor@5bb79d4f counterGroup:{ name:null counters:{} } }} channels:{mem_channel=org.apache.flume.channel.MemoryChannel{name: mem_channel}} }
18/05/06 07:00:13 INFO node.Application: Starting Channel mem_channel
18/05/06 07:00:13 INFO instrumentation.MonitoredCounterGroup: Monitored counter group for type: CHANNEL, name: mem_channel: Successfully registered new MBean.
18/05/06 07:00:13 INFO instrumentation.MonitoredCounterGroup: Component type: CHANNEL, name: mem_channel started
18/05/06 07:00:13 INFO node.Application: Starting Sink stdout_sink
18/05/06 07:00:13 INFO node.Application: Starting Source nc_source
18/05/06 07:00:13 INFO source.NetcatSource: Source starting
18/05/06 07:00:13 INFO source.NetcatSource: Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/127.0.0.1:44444]
~~~

### Step-4: Verify flume agent

* **Send data using netcat**

~~~
asus@asus-GL553VD:~$ telnet 192.168.211.142 44444
Trying 192.168.211.142...
Connected to 192.168.211.142.
Escape character is '^]'.
shreyash
OK
I am trying to send data to netcat server started by flume agent
OK
Hi
OK
Hello World
OK
Again
OK
Testing
OK
~~~

* **Verify data on standard output of flume agent**

~~~
18/05/06 07:06:33 INFO node.Application: Starting Channel mem_channel
18/05/06 07:06:33 INFO instrumentation.MonitoredCounterGroup: Monitored counter group for type: CHANNEL, name: mem_channel: Successfully registered new MBean.
18/05/06 07:06:33 INFO instrumentation.MonitoredCounterGroup: Component type: CHANNEL, name: mem_channel started
18/05/06 07:06:33 INFO node.Application: Starting Sink stdout_sink
18/05/06 07:06:33 INFO node.Application: Starting Source nc_source
18/05/06 07:06:33 INFO source.NetcatSource: Source starting
18/05/06 07:06:33 INFO source.NetcatSource: Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/192.168.211.142:44444]
18/05/06 07:06:44 INFO sink.LoggerSink: Event: { headers:{} body: 73 68 72 65 79 61 73 68 0D                      shreyash. }
18/05/06 07:07:08 INFO sink.LoggerSink: Event: { headers:{} body: 49 20 61 6D 20 74 72 79 69 6E 67 20 74 6F 20 73 I am trying to s }
18/05/06 07:07:19 INFO sink.LoggerSink: Event: { headers:{} body: 48 69 0D                                        Hi. }
18/05/06 07:07:24 INFO sink.LoggerSink: Event: { headers:{} body: 48 65 6C 6C 6F 20 57 6F 72 6C 64 0D             Hello World. }
18/05/06 07:07:39 INFO sink.LoggerSink: Event: { headers:{} body: 41 67 61 69 6E 0D                               Again. }
18/05/06 07:07:43 INFO sink.LoggerSink: Event: { headers:{} body: 54 65 73 74 69 6E 67 0D                         Testing. }
~~~