## Introduction

### Overview
* Apache Flume is a distributed, reliable, and available system for efficiently collecting, aggregating and moving large amounts of log data from many different sources to a centralized data store
* The use of Apache Flume is not only restricted to log data aggregation. Since data sources are customizable, Flume can be used to transport massive quantities of event data including but not limited to network traffic data, social-media-generated data, email messages and pretty much any data source possible 
* A Flume agent is a (JVM) process that hosts the components through which events flow from an external source to the next destination (hop). Flume agent consists of following components:
   * Source : It is reponsible to read data from external source and write it to channel
   * Channel : It is reponsible to hold data between source and sink
   * Sink : It is responsible to read data from channel to write it to external target

### Architecture
Flume supports different types of data flow models:

* **(1) Typical Data Flow**
  ![Alt text](_images/_1_typical_data_flow.png?raw=true "Typical Data Flow")  

* **(2) Multi Agent Flow**
  ![Alt text](_images/_2_multi_agent_flow.png?raw=true "Multi Agent Flow")  

* **(3) Consolidated Flow**
  ![Alt text](_images/_3_consolidated_flow.png?raw=true "Consolidaed Flow")  

* **(4) Multiplexing Flow**
  ![Alt text](_images/_4_multiplexing_data_flow.png?raw=true "Multiplexing Data Flow")  
