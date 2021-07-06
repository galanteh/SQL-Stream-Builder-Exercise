# SQL Stream Builder Exercise
This exercise is based on the [Edge2Ai Streaming Exercise](https://github.com/asdaraujo/edge2ai-workshop/blob/trunk/sql_stream_builder.adoc)

**NOTE:** This lab assumes that the [From Edge to Streams Processing](https://github.com/asdaraujo/edge2ai-workshop/blob/trunk/streaming.adoc) lab has been completed. If you haven't done so, please ask your instructor to set your cluster state for you so that you can perform the steps in this lab (or you can do this yourself by SSH'ing to your cluster host and running the script `/tmp/resources/reset-to-lab.sh 9`) 

In this workshop you will use SQL Stream Builder to query and manipulate data streams using SQL language. SQL Stream Builder is a powerful service that enables you to create Flink jobs without having to write Java/Scala code.

# Labs summary
* [Introduction](#Introduction)
* [Lab 1 - Create a Source Virtual Table for a topic with JSON messages](#VirtualTable)
* [Lab 2 - Run a simple query](#Query)
* [Lab 3 - Doing a Transformation from the table](#Transformation)
* [Lab 4 - Setting the Consumer Group to the table](#Settings)
* [Lab 5 - Computing and storing agregation results](#Agregation)

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

1. Go back to the tab **compose**, and in the Text are you can type your SQL. In our case, we will just use a simple SQL and then we will chooose **execute**.

```sql
Select * from iot_enriched_json
```
Tip: If you don't want to type the name of the table, just click CTRL+SPACE and a menu will be shown to select your table name.

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image8.png)

2. Scroll to the bottom of the page and you will see the log messages generated by your query execution.

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image10.png)

3. After a few seconds the SQL Console will start showing the results of the query coming from the **iot_enriched topic**.
The data displayed on the screen is only a sample of the data returned by the query, not the full data.

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image11.png)

<a name="Transformation"></a>
# Lab 3 - Doing a Transformation from the table
In this lab, you will create a new table to run a transformation in one column, so you can have in the column *sensor_ts* a value showed in milliseconds instead of microseconds

1. Let's edit our virtual table

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image12.png)

2. Go to the tab "Transformation"

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image13.png)

3. In that eempty text are of transformation, please, introduce the following javascript code:

```javascript
var payload = JSON.parse(record.value);
payload['sensor_ts'] = Math.round(payload['sensor_ts']/1000);
delete payload['response'];
JSON.stringify(payload);
```
This code will read the record value into a variable called payload. In that payload, you will have a dictionary with all the key and values. Code will transform the milliseconds (16 digits) into microseconds (13 digits) dividing the value by 1000 and round it up.
After this operation it will remove a key called response which is the answer of the machine learning model but it's also in the is_healthy field.
In the end of the code, we will transform all the values into a Json again. 

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image14.png)

4. Detect the new schema. After saving the changes in the transformation tab, now the table description of the fields has been changed. So, the easy way is the re-run the detection of the schema and save once the again the new changes.

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image15.png)

5. Re-run your query

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image16.png)

6. Check the results

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image17.png)

<a name="Settings"></a>
# Lab 4 - Setting the Consumer Group to the table

Setting the Consumer Group properties for a virtual table will ensure that if you stop a query and restart it later, the second query execute will continue to read the data from the point where the first query stopped, without skipping data. However, if multiple queries use the same virtual table, setting this property will effectively distribute the data across the queries so that each record is only read by a single query. If you want to share a virtual table with multiple distinct queries, ensure that the Consumer Group property is unset.

1. Edit the table
2. Click on the Properties tab, enter the following value for the Consumer Group property and click Save changes.
```
Consumer Group: ssb-iot-1
```
![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image18.png)

<a name="Agregation"></a>
# Lab 5 - Computing and storing agregation results
We want to start computing window aggregates for our incoming data stream and make the aggregation results available for downstream applications. SQL Stream Builder’s Sink Virtual Tables give us the ability to publish/store streaming data to several different services (Kafka, AWS S3, Google GCS, Elastic Search and generic webhooks). In this lab we’ll use a Kafka sink to publish the results of our aggregation to another Kafka topic.

