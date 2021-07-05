# SQL Stream Builder Exercise
This exercise is based on the [Edge2Ai Streaming Exercise](https://github.com/asdaraujo/edge2ai-workshop/blob/trunk/sql_stream_builder.adoc)

**NOTE:** This lab assumes that the [From Edge to Streams Processing](https://github.com/asdaraujo/edge2ai-workshop/blob/trunk/streaming.adoc) lab has been completed. If you haven't done so, please ask your instructor to set your cluster state for you so that you can perform the steps in this lab (or you can do this yourself by SSH'ing to your cluster host and running the script `/tmp/resources/reset-to-lab.sh 9`) 

In this workshop you will use SQL Stream Builder to query and manipulate data streams using SQL language. SQL Stream Builder is a powerful service that enables you to create Flink jobs without having to write Java/Scala code.

# Labs summary
* [Introduction](#Introduction)
* [Lab 1 - Create a Data Source](#DataSource)
* Lab 2 - Create a Source Virtual Table for a topic with JSON messages
* Lab 3 - Create a Source Virtual Table for a topic with AVRO messages
* Lab 4 - Run a simple query
* Lab 5 - Computing and storing agregation results

# Introduction
<a name="Introduction"></a>
In this lab, and the subsequent ones, we will use the iot_enriched topic created and populated in previous labs and contains a datastream of computer performance data points.
So let’s start with a straightforward goal: to query the contents of the iot_enriched topic using SQL to examine the data that is being streamed.
Albeit simple, this task will show the ease of use and power of SQL Stream Builder (SSB).

<a name="DataSource"></a>
# Lab 1 - Create a Data Source
Before we can start querying data from Kafka topics we need to register the Kafka clusters as data sources in SSB.

1. On the Cloudera Manager console, click on the Cloudera logo at the top-left corner to ensure you are at the home page and then click on the SQL Stream Builder service.

2. Click on the SQLStreamBuilder Console link to open the SSB UI.

3. On the logon screen, authenticate with user **admin** and password **supersecret1**

3. You will notice that SSB already has a Kafka cluster registered as a data source, named CDP Kafka. This source is created automatically for SSB when it is installed on a cluster that also has a Kafka service:





