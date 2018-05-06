## Introduction

### Overview

* Apache Flume is a distributed, reliable, and available system for efficiently collecting, aggregating and moving large amounts of log data from many different sources to a centralized data store

* The use of Apache Flume is not only restricted to log data aggregation. Since data sources are customizable, Flume can be used to transport massive quantities of event data including but not limited to network traffic data, social-media-generated data, email messages and pretty much any data source possible

* In Hadoop echo system, Apache Flume plays important role to ingest data from different sources to HDFS  

### Architecture
* A Flume agent is a (JVM) process that hosts the components through which events flow from an external source to the next destination (hop). Flume agent consists of following components:
  * Source : It is reponsible to read data from external source and write it to channel
  * Channel : It is reponsible to hold data between source and sink
  * Sink : It is responsible to read data from channel tand o write it to external target

   
  ![Alt text](_images/_1_typical_data_flow.png?raw=true "Typical Data Flow Model")


### Components

* Below are few useful components provided by flume out of the box:
  
  * **Flume Sources**
    * Netcat Source
    * Exec Source
    * JMS Source
    * Kafka Source
    * HTTP Source
    * Avro Source
    * Thrift Source

  * **Flume Channels**
    * Memory Channel
    * File Channel
    * JDBC Channel
    * Kafka Channel

  * **Flume Sinks**
    * Logger Sink
    * HDFS Sink
    * Kafka Sink
    * HTTP Sink
    * Avro Sink
    * Thrift Sink

### Data Flow Models

* **Multi Agent Flow** : In order to flow the data across multiple agents, the sink of the previous agent and source of the current agent needs to be avro type with the sink pointing to the hostname (or IP address) and port of the source.

  
  ![Alt text](_images/_2_multi_agent_flow.png?raw=true "Multi Agent Data Flow")


* **Consolidated Flow** : A very common scenario in log collection is a large number of log producing clients sending data to a few consumer agents that are attached to the storage subsystem. For example, logs collected from hundreds of web servers sent to a dozen of agents that write to HDFS cluster.
  
  
  ![Alt text](_images/_3_consolidated_flow.png?raw=true "Consolidaed Data Flow")


* **Multiplexing Flow** : Flume supports multiplexing the event flow to one or more destinations. This is achieved by defining a flow multiplexer that can replicate or selectively route an event to one or more channels.

  
  ![Alt text](_images/_4_multiplexing_data_flow.png?raw=true "Multiplexing Data Flow")  


## Setup

* Cloudera QuickStart VM should be up & running (Click [here](https://github.com/124938/learning-hadoop-vendors/tree/master/cloudera/_1_quickstart_vm/README.md) to know more details on it) OR 

* Any other hadoop cluster should be available with flume configured inside

## Verification
 
### Step-1: Login to Quick Start VM OR Gateway Node of hadoop cluster

~~~
asus@asus-GL553VD:~$ ssh cloudera@192.168.211.142
cloudera@192.168.211.142's password: 
Last login: Sun May  6 04:51:45 2018 from 192.168.211.1

[cloudera@quickstart ~]$ 
~~~

### Step-2: Verification of flume binaries

~~~
[cloudera@quickstart ~]$ cd /usr/lib/flume-ng/bin/
[cloudera@quickstart bin]$ ls -ltr
total 16
-rwxr-xr-x 1 root root 12628 Jun 29  2017 flume-ng
~~~

### Step-3: Verification of flume configurations

~~~
[cloudera@quickstart bin]$ cd /etc/flume-ng/conf
[cloudera@quickstart conf]$ ls -ltr
total 16
-rw-r--r-- 1 root root 1565 Jun 29  2017 flume-env.sh.template
-rw-r--r-- 1 root root 1455 Jun 29  2017 flume-env.ps1.template
-rw-r--r-- 1 root root 1661 Jun 29  2017 flume-conf.properties.template
-rw-r--r-- 1 root root 3118 Jun 29  2017 log4j.properties
-rw-r--r-- 1 root root    0 Jun 29  2017 flume.conf
~~~

### Step-4: Verification of flume command

~~~
[cloudera@quickstart conf]$ flume-ng help
Usage: /usr/lib/flume-ng/bin/flume-ng <command> [options]...

commands:
  help                      display this help text
  agent                     run a Flume agent
  avro-client               run an avro Flume client
  version                   show Flume version info

global options:
  --conf,-c <conf>          use configs in <conf> directory
  --classpath,-C <cp>       append to the classpath
  --dryrun,-d               do not actually start Flume, just print the command
  --plugins-path <dirs>     colon-separated list of plugins.d directories. See the
                            plugins.d section in the user guide for more details.
                            Default: $FLUME_HOME/plugins.d
  -Dproperty=value          sets a Java system property value
  -Xproperty=value          sets a Java -X option

agent options:
  --name,-n <name>          the name of this agent (required)
  --conf-file,-f <file>     specify a config file (required if -z missing)
  --zkConnString,-z <str>   specify the ZooKeeper connection to use (required if -f missing)
  --zkBasePath,-p <path>    specify the base path in ZooKeeper for agent configs
  --no-reload-conf          do not reload config file if changed
  --help,-h                 display help text

avro-client options:
  --rpcProps,-P <file>   RPC client properties file with server connection params
  --host,-H <host>       hostname to which events will be sent
  --port,-p <port>       port of the avro source
  --dirname <dir>        directory to stream to avro source
  --filename,-F <file>   text file to stream to avro source (default: std input)
  --headerFile,-R <file> File containing event headers as key/value pairs on each new line
  --help,-h              display help text

  Either --rpcProps or both --host and --port must be specified.

Note that if <conf> directory is specified, then it is always included first
in the classpath.
~~~

~~~
[cloudera@quickstart conf]$ flume-ng version
Flume 1.6.0-cdh5.12.0
Source code repository: https://git-wip-us.apache.org/repos/asf/flume.git
Revision: 4d37abdc65a53a0af1246de51bdcf1d8557669eb
Compiled by jenkins on Thu Jun 29 04:37:39 PDT 2017
From source with checksum af8d398eddf3e3b4ddbada14895722ac
~~~

