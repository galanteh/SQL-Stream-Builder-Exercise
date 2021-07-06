# Cloudera SQL Stream Builder Exercise
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
* [Lab 6 - Materialized Views](#MV)

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

1. Let’s first create a topic (**sensor6_stats**) where to publish our aggregation results:

    1. Navigate to the SMM UI (**Cloudera Manager > SMM service > Streams Messaging Manager Web UI**).
    
    2. On the SMM UI, click the **Topics tab** (topics icon).

    3. Click the **Add New** button.

    ![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image19.png)

    4. Enter the following details for the topic and click Save when ready:

        * Topic name: **sensor6_stats**

        * Partitions: **10**

        * Availability: **Low**

        * Cleanup Policy: **delete**

        ![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image20.png)

2. To create the Sink Virtual Table, click on **Console (on the left bar) > Tables > Add Table > Apache Kafka**.

3. In the new dialog, enter
    1. Table Name: **sensor6_stats_sink**
    2. Kafka Cluster: **CDP Kafka**
    3. Topic Name: **sensor6_stats**
    4. Data Format: **JSON**
    5. Schema Definition: Check **Dynamic Schema**

    ![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image21.png)

4. Save the new table and you will see a list of tables like this:

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image22.png)

5. On the SSB UI, click on Console (on the left bar) > **Compose > SQL and type the query shown below.**

    This query will compute aggregates over 30-seconds windows that slide forward every second. For a specific sensor value in the record (sensor_6) it computes the following aggregations for each window:

    * Number of events received

    * Sum of the sensor_6 value for all the events

    * Average of the sensor_6 value across all the events

    * Min and max values of the sensor_6 field

    * Number of events for which the sensor_6 value exceeds 70

    ```sql
    SELECT
        sensor_id as device_id,
        HOP_END(eventTimestamp, INTERVAL '1' SECOND, INTERVAL '5' MINUTE) as windowEnd,
        count(*) as sensorCount,
        sum(sensor_6) as sensorSum,
        avg(cast(sensor_6 as float)) as sensorAverage,
        min(sensor_6) as sensorMin,
        max(sensor_6) as sensorMax,
        sum(case when sensor_6 > 70 then 1 else 0 end) as sensorGreaterThan60
   FROM iot_enriched_json
   GROUP BY
        sensor_id,
        HOP(eventTimestamp, INTERVAL '1' SECOND, INTERVAL '5' MINUTE)
    ```
  
![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image23.png)
 
6. Wait looking at the logs about the process. If everything runs as expected, you will see in the logs and after that you will see the results automatically. 

Logs:

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image24.png)

Results:

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image25.png)

7. You can check the SQL jobs running in the **Console (on the left bar) > SQL Jobs** tab.

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image26.png)

9. If you click in the Flink dashboad, you can check the process at Flink. 

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image27.png)

10. Finally, you will see the new data being sink in the topic **sensor6_stats** in the SMM console.

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image28.png)

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image29.png)

<a name="MV"></a>
# Lab 6 - Materialized Views
SQL Stream Builder can also take keyed snapshots of the data stream and make that available through a REST interface in the form of Materialized Views. In this lab we’ll create and query Materialized Views (MV).
We will define MVs on top of the query we created in the previous lab. Make sure that query is running before executing the steps below.

1. On the **Console_ > SQL Jobs** tab, verify that the **sensor_6_sink** job is running. Select the job and click on the Edit Selected Job button.

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image30.png)

2. Select the Materialized View tab for that job and set the following values for the its properties:

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image31.png)

3. Choose the elements like in the image below.

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image32.png)

4. To create a MV we need to have an API Key. The API key is the information given to clients so that they can access the MVs. If you have multiple MVs and want them to be accessed by different clients you can have multiple API keys to control access.

If you have already created an API Key in SSB you can select it from the drop-down list. Otherwise, create one on the spot by clicking on the Add API Key button shown above. Use **sensor_6_sink** as the Key Name for example.

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image33.png)

5. Click **Apply Configuration**. This will enable the **Add Query** button below.

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image34.png)

![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image35.png)

6. Click **Add Query** to create a new MV. It is possible to specify parameters for a MV so that you can filter the contents at query time. For example, we will create a MV that allows filtering by specifying a range of values for the sensorAverage column.

    * URL Pattern:   **above60withRange/{lowerTemp}/{upperTemp}**
    * Query Builder: **<click "Select All" to add all columns>**
    * Filters:       
        * sensorGreatThan60  greater 0
            * AND
         * sensorAverage greater or equal {lowerTemp}
            * AND
         * sensorAverage less or equal {upperTemp}

    ![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image36.png)
    
7. After that, you will notice that the new URL for this MV has placeholders for the {lowerTemp} and {upperTemp} parameters.
8. opy the MV URL to a text editor and replace the placeholders with actual values for those parameters.
The example below shows a filter for sensorAverage values between 80 and 85, inclusive:
    ```
    .../above60withRange/50/70?key=...
    ```
    
    ![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image37.png)    
    
9. After replacing the values, open the URL on your web browser to retrieve the filtered data.

    ![](https://github.com/galanteh/SQL-Stream-Builder-Exercise/blob/main/images/image38.png)














