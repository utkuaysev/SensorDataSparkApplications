This project was split into three parts. In the first part, we simulate real-time data using Spark Streaming that gets stored into HBase. In the second part, we save a CSV data file to Hive and perform queries on it using Spark SQL. We have data visualizations for both these parts using Plotly on a Jupyter Notebook reading data from HBase, and Tableau for Hive data. Our third part is an independent Kafka exploration performing a Word Count on some test input, provided in the body of a Postman message.

![diagram-bdt](https://github.com/utkuaysev/SparkSensorStreaming/assets/33395066/1463ed93-6bfe-4947-aed8-79fcd9e8d6bc)

For Spark parts, cloudera's centos7 vm is used. This vm comes with necessary jar files for hadoop, spark, and hive. 

## Spark Stream with Java

This repository processes data that includes temperature in degrees Celsius,
humidity percentages, and ping times for sensor data originating from a room.
It divides the home sensor dataset into chunks and generates new files.
The Spark Streaming job is configured to monitor new file creations and subsequently applies an algorithm to identify
data surpassing a certain threshold.
The identified data is then stored in HBase, along with timestamped records of temperature, humidity, and sensor ping data.

### To run the hbase:
```
sudo service hbase-master start
sudo service hbase-regionserver start
hbase shell
```
### To run the project as it is:
- Run HBaseSaver.java program in Eclipse to create HBase table sensor_data. If this data exists it will delete it and create it again.
- Run Streaming.java program in Eclipse to start listening to new file creations. This application will process the data in 15 second intervals. Then, it will check the threshold values for temperature and humidity values, which are 17 celsius degrees for temperature and 70 percent for humidity. Also this program will save the data which is above the threshold to hbase.
- Run FileSplitter.java program in an external console using the commands:
```
 javac FileSplitter.java
 java FileSplitter
```
- Necessary jar files needs to be imported

### HBaseSaver Output
<img width="834" alt="hbaseSaver" src="https://github.com/utkuaysev/SensorDataSparkApplications/assets/33395066/e9a18f32-7411-4967-928d-1c6a11973c89">

### FileSplitter Output
<img width="251" alt="MicrosoftTeams-image (1)" src="https://github.com/utkuaysev/SensorDataSparkApplications/assets/33395066/da97bb2f-f54c-4b1f-af9e-1048d9c59720">
<img width="178" alt="MicrosoftTeams-image (2)" src="https://github.com/utkuaysev/SensorDataSparkApplications/assets/33395066/93b6ebe8-c2ea-4f44-b5fb-78193e2d0e04">
<img width="176" alt="MicrosoftTeams-image (3)" src="https://github.com/utkuaysev/SensorDataSparkApplications/assets/33395066/8663bcae-dbac-416b-b0df-d4e8aee87972">

### Streamer Output
![image](https://github.com/utkuaysev/SensorDataSparkApplications/assets/33395066/65b71fdd-c7b5-41af-8aea-48cbcc2fe922)


### HBase
<img width="398" alt="MicrosoftTeams-image" src="https://github.com/utkuaysev/SensorDataSparkApplications/assets/33395066/27e78903-0322-4e3c-a04f-1aabed9a044e">


## Hive with SparkSQL
This part reads some static data from a CSV file and puts the data into Hive database with the name “project” and table name “weather”, then performs some data manipulations to create two new tables (avgPerDay and avgPerMonth) to get the average temperature and humidity per day and month respectively.

### To run the project as it is
- Give write permissions to the folder "/tmp/hive" using ```sudo chmod 777 /tmp/hive```.
- Add the project to Eclipse.
- Import SparkSQL and Hive jars.
- You can change "spark.sql.warehouse.dir"; otherwise, it will be saved to HDFS in "/user/hive/warehouse".
- You can change the value of setMaster to yarn if you need to use yarn ResourceManager; otherwise, it will just use the local machine.
- Run SparkHiveCreateTable from Eclipse to first read a CSV file and save it as a Hive table.
- Run SparkHiveManipulations if you want to perform some analysis on data and create new Hive tables.
- Run SparkHiveTesting so you can see the table output on your console or check the location using HUE.

#### Creating database project and table weather
![image](https://github.com/utkuaysev/SensorDataSparkApplications/assets/20099573/794c8929-a5ee-40c4-8676-1d46215f3193)
![image](https://github.com/utkuaysev/SensorDataSparkApplications/assets/20099573/c5e50117-4761-4749-b306-d241263a093a)

#### Performing data manipulations and creating tables avgPerDay and avgPerMonth
![image](https://github.com/utkuaysev/SensorDataSparkApplications/assets/20099573/432a110c-5981-47d0-9173-761dc4d1da6d)
![image](https://github.com/utkuaysev/SensorDataSparkApplications/assets/20099573/a38957fb-80bc-44e5-9716-500d035bfc10)


## Visualization
### Jupyter Notebook
- https://linuxhostsupport.com/blog/how-to-install-jupyter-on-centos-7/
```
hbase thrift start
pip install happybase
pip install plotly
pip install pandas
```
- Connect to HBase from jupyter, transform the data and use on the plotly.
### Tableau connected to Apache Hive
- Install Tableau on local machine
- Install Cloudera Hive ODBC Driver to connect Tableau to Hive Server (https://www.cloudera.com/downloads/connectors/hive/odbc/2-7-0.html)
- In Tableau, under “Connect to…” under “Server” select “Cloudera Hadoop” which is listed under “Installed Connectors”
- If the driver is successfully installed, a screen to populate connection details appears. The connection is “HiveServer2”, the server address (can be found by using the shell command ifconfig on our VM, as “eth4 inet addr:”, port 10000 which is the default Hive port used by Cloudera, username authentication with username “cloudera” [and no password!]). 
- Once signed in, select appropriate schema and table. 
- Specific calculated field in Tableau: ``` AVG(AvgTemp [or AvgHumidity] GROUPBY month_id) ```

## Kafka with Java
- Using some sample data of sentences with/without duplicates, the Kafka distributed messaging system produces a topic, which contains the data needed to be consumed by the consumer. 
- The consumer then consumes the message and performs logical computation and provides the overall count for each output.
- With two applications, one being the producer and the other being the consumer, a rest api is used to correctly provide the output.

### To use Kafka properly
- Download two Spring Boot applications through a maven project.
- Add the project to eclipse, by adding them as a maven project.
- Open the command line and run apache zookeeper
- Open the command line and run kafka server
- Run both the producer and consumer applications in intellij.
- Open postman and set the data using Post API.
- Data should successfully display in the console.

#### Zookeeper & Kafka commands
 ```
 .\bin\windows\zookeeper-server-start.bat .\config\zookeeper.properties
 .\bin\windows\kafka-server-start.bat .\config\server.properties
```
#### Postman command: 
When you open postman click on new, and click on HTTP.
In the front you should see something called GET, change it to POST and type the URL: http://localhost:8080/word/post
After doing so in postman, click on raw, and change the format to json, and post this in the body:
```
{

    "message": "In a well-designed database, information is stored in tables with fields such as name, age, and address; each table is designed according to a schema that defines the relationships between different entities and their attributes, and queries are used to extract specific information from the database, allowing users to filter and sort records based on various criteria, while indexes play a crucial role in optimizing the speed of data retrieval by providing quick access to particular rows in a table."
}
```
Once you run both applications, click 'send' on postman, and the output should be displayed in intellij console.

