# Realtime Log Analysis

## What is Log Analysis?
The process of evaluating, understanding, and comprehending computer-generated documents known as logs is known as log analysis. A wide range of programmable technologies, including networking devices, operating systems, apps, and more, produce logs. A log is a collection of messages in chronological order that describe what is going on in a system. Log files can be broadcast to a log collector over an active network or saved in files for later analysis. Regardless, log analysis is the subtle technique of evaluating and interpreting these messages in order to get insights into any system's underlying functioning. Web server log analysis can offer important insights on everything from security to customer service to SEO. The information collected in web server logs can help you with:
    • Network troubleshooting efforts
    • Development and quality assurance
    • Identifying and understanding security issues
    • Customer service
    • Maintaining compliance with both government and corporate policies

## Usage of Dataset:
Here we are going to use NASA access log data in the following ways:
- Extraction: During the extraction process, the downloaded dataset from Kaggle is
ingested using NiFi processors and connections. The data is streamed from the data
file using NiFi followed by the creation of topics and publishing logs using Apache
Kafka.
- Transformation and Load: During the transformation and load process, we read data
from Apache Kafka as streaming Dataframe according to schema creation with
extraction and cleansing of log data and loading to Cassandra for Speed layer and HDFS for Batch layer. Then data is visualized using Plotly in Dash.

## Tech Stack

Docker  
Jupyter Lab  
Spark Structured Streaming  
NiFi  
Kafka  
Python  
HDFS  
Cassandra  
Plotly  
Dash  

## Hands-On

### Get env ready

Clone this directory and `cd` into it  

`cd KafkaNifiSparkStreaming_LogAnalysis`  

Get the environment up and ready.  

`docker-compose up -d`

`docker ps`  to check if all is up and running

### Log processing

Download data from `https://drive.google.com/file/d/11MtpPlaaBd3oC4IKXlONBuDI8nKq0E6a/view?usp=sharing` and add it the the `data`, make sure it named `data.csv` folder

Copy data from `data` directory to the container `jupyterlab` using:  
`docker cp -r data/* jupyterlab:/opt/workspace/.`

Now go to Jupyterlab at `http://localhost:4888/lab`

Execute the code in the `log_preprocessing.ipynb` (in jupyterlab) file step by step to have an understanding on how log data can be process

### Extration Nifi kafka

connect to nifi container and run the commands below
`docker exec -i -t nifi bash`  
`mkdir -p nasa_logs && cp /opt/workspace/nifi/data.csv nasa_logs/data.csv`

Create Kafka topic by connecting to Kafka container `docker exec -it kafka bash`
and execute the following

`docker exec -it kafka bash` enter into kafka CLI

`kafka-topics.sh --create --topic nasa_logs_demo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181` creation of topic named `nasa_logs_demo`

Some others useful Kafka commands to know

`kafka-topics.sh --list --bootstrap-server localhost:29092` list topics

`kafka-topics.sh --describe --topic nasa_logs_demo --zookeeper zookeeper:2181` describe topic

`kafka-topics.sh --delete --topic nasa_logs_demo --zookeeper zookeeper:2181` delete topic


#### Streaming data from file system using NiFi
Go to http://localhost:2080/nifi/  

Nifi Setup:  
Using the GetFile, SplitText and PublishKafka processors make a Nifi flow to get the data from  `/opt/nifi/nifi-current/nasa_logs` directory in the nifi container to the Kafka topic `nasa_logs_demo`

Tips: To avoid memory-related issues he SplitText may be used several time decreasing the split size each time. If you process all the lines at once, the flow file in the memory may exceed the available memory, and thus the process runs into issues. (You can import the template `log_to_kafka_template.xml` to see a sample of solution)

Check if the data is in Kafka:  
`kafka-console-consumer.sh --bootstrap-server localhost:29092 --topic nasa_logs_demo --from-beginning --max-messages 30` consume/read data from topic

### Transformation and load to Cassandra and Hdfs

#### Create namespace and table in Cassandra

CQL Commands
  `docker exec -i -t cassandra bash`
  `cqlsh -u cassandra -p cassandra`
  `CREATE KEYSPACE IF NOT EXISTS LogAnalysis WITH replication = {'class':'SimpleStrategy', 'replication_factor':1};`
  `CREATE TABLE IF NOT EXISTS LogAnalysis.NASALog (host text , time text , method text , url text , response text , bytes text, extension text, time_added text,PRIMARY KEY (host));`

Some cassandra commands (Find out by yourself what they do):  
    `show version;`  
    `desc keyspaces;`  
    `desc keyspace_name;`  
    `use keyspace_name;`  
    `describe tables;`  
    `desc table_name;` 
    `truncate table LogAnalysis.NASALog;`  
    `select * from loganalysis.nasalog;`  

#### Create a folder in HDFS

  `docker exec -i -t namenode bash`  
  `hdfs dfs -mkdir -p /output/nasa_logs/`  
   connect to `http://localhost:50070/` to have a glimpse

#### Read Streaming Data and Cleansing

Now go to Jupyterlab at `http://localhost:4888/lab`  

Execute the code in the `log_listener.ipynb` (in jupyterlab) file step by step to have an understanding on how log data can be process

## Visualization

Dash Application at: `http://localhost:8050/`  
(this will be available when executed `log_visualizer.ipynb`)