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


