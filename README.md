# SQL Stream Builder Exercise
This exercise is based on the [Edge2Ai Streaming Exercise](https://github.com/asdaraujo/edge2ai-workshop/blob/trunk/sql_stream_builder.adoc)

**NOTE:** This lab assumes that the [From Edge to Streams Processing](https://github.com/asdaraujo/edge2ai-workshop/blob/trunk/streaming.adoc) lab has been completed. If you haven't done so, please ask your instructor to set your cluster state for you so that you can perform the steps in this lab (or you can do this yourself by SSH'ing to your cluster host and running the script `/tmp/resources/reset-to-lab.sh 9`) 

In this workshop you will use SQL Stream Builder to query and manipulate data streams using SQL language. SQL Stream Builder is a powerful service that enables you to create Flink jobs without having to write Java/Scala code.

# Labs summary
* [Introduction](#Introduction)
* [Lab 1 - Create a Source Virtual Table for a topic with JSON messages](#VirtualTable)
* [Lab 2 - Run a simple query](#Query)
* Lab 3 - Computing and storing agregation results

# Introduction
<a name="Introduction"></a>
In this lab, and the subsequent ones, we will use the iot_enriched topic created and populated in previous labs and contains a datastream of computer performance data points.
So let’s start with a straightforward goal: to query the contents of the iot_enriched topic using SQL to examine the data that is being streamed.
Albeit simple, this task will show the ease of use and power of SQL Stream Builder (SSB).

<a name="VirtualTable"></a>
# Lab 1 - Create a Source Virtual Table for a topic with JSON messages
Before we can start querying data from Kafka topics we need to register the Kafka clusters as data sources in SSB.

1. On the Cloudera Manager console, click on the Cloudera logo at the top-left corner to ensure you are at the home page and then click on the SQL Stream Builder service.

2. Click on the SQLStreamBuilder Console link to open the SSB UI.

3. On the logon screen, authenticate with user **admin** and password **supersecret1**

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image1.png)

3. Go to the **tables tab** and you will add a new table.

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image2.png)

4. You will choose the first option on the menu, Apache Kafka.

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image3.png)

5. You will notice that SSB already has a Kafka cluster registered as a data source, named CDP Kafka. This source is created automatically for SSB when it is installed on a cluster that also has a Kafka service:

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image4.png)

6. You will create a table over the topic **iot_enriched**. This topic has simulation from the IoT with a field called **is_healthy** and **response** which is the prediction comming from the Machine Learning model. 

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image5.png)

7. You will need to provide an schema so the the JSON can be interpreted. In our case, we just can let it to the SBS to **detect the schema in automatic** using the "Detect Schema" button

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image6.png)

8. Save changes and you will have created your first SQL table over Kafka. 

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image7.png)

<a name="Query"></a>
# Lab 2 - Run a simple query
Now that you have table available, we can run now a query. 

1. Go back to the tab **compose**, and in the Text are you can type your SQL. In our case, we will just use a simple 

```sql
Select * from iot_enriched_json
```
Tip: If you don't want to type the name of the table, just click CTRL+SPACE and a menu will be shown to select your table name.

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image8.png)




