# Hadoop Project for Beginners: SQL Analytics with Hive, Spark/Scala

## AIm
To perform Hive analytics on Customer Demographics data using big data tools such as
Sqoop, Spark, and HDFS.

## Data Description
Adventure Works is a free sample database of retail sales data. In this project, we will
be only using Customer test, Individual test, and Credit card tables from this database.
Customer test table contains data like Customer ID, Territory ID, Account number,
Customer Type etc. Individual test table contains data like Customer ID, Contact ID and
Demographics. Credit card table contains data like Credit card ID, Card type, Card
number, Expiry month, Expiry year.

## Tech Stack
Language: SQL, Scala  
Services: Docker, MySQL, Sqoop, Hive, HDFS, Spark

## Approach
Create tables in MySQL.  
Load data from MySQL into HDFS storage using Sqoop commands.  
Move data from HDFS to Hive.  
Integrate Hive into Spark.  
Using scala programming language, extract Customer demographics information
from data and store it as parquet files.  
Move parquet files from Spark to Hive.  
Create tables in Hive and load data from Parquet files into tables.  
Perform Hive analytics on Customer demographics data.  

## Hand-on lab

Clone this repo and cd into the cloned directory.  

cd SqoopHiveSparkMysql  
-f Code/docker_exp/docker-compose.yml up -d  

docker cp Code/Docker-Hive-Spark-Dependencies/postgresql-42.3.1.jar hdp_spark-master:/spark/jars/  

docker cp ra_hive-server:/opt/hive/conf/hive-site.xml .  
docker cp hive-site.xml hdp_spark-master:/spark/conf/  
rm hive-site.xml  
docker cp Code/ ra_mysql:/opt/Code  
docker cp Code/ ra_hive-server:/opt/  

1. Refer 01_partial_dataset_creation.sql contains MYsql table creation commands for partial dataset and view for importing data (I suggest run the code by copy pasting from the file).

docker exec -i -t ra_mysql bash  
mysql -u root -p  
password: example  
mysql> source /opt/Code/01_partial_dataset_creation.sql;  

2. Refer 03_Sqoop-import-commands.txt , contains sqoop import commands to import data from Mysql database.  

docker exec -i -t ra_sqoop bash  

sqoop import --connect jdbc:mysql://ra_mysql:3306/testdb --username root \  
--target-dir /input/customerdemo/ \  
--query 'SELECT CustomerID, AccountNumber, CustomerType, Demographics, TerritoryID, ModifiedDate FROM v_customer_demo WHERE $CONDITIONS' \  
--split-by CustomerID --password example  

sqoop import --connect jdbc:mysql://ra_mysql:3306/testdb --username root --password example \  
--query 'SELECT CreditCardID, CardType, CardNumber, ExpMonth, ExpYear, ModifiedDate FROM ccard WHERE $CONDITIONS' \  
--split-by CreditCardID --target-dir /input/creditcard/  

Commands to check the files created  

hdfs dfs -ls /input/customerdemo/  
hdfs dfs -ls /input/creditcard/   

4. Refer 04_Hive_tables_creation.hql , Contains HIVE tables create statements.  

docker exec -i -t ra_hive-server bash  
hive  
source /opt/Code/04_Hive_tables_creation.hql;  

5. Refer 06_customer_demographic.scala , this is used to fetch demographic XML values into Parquet files.  

docker cp /Code/06_customer_demographic.scala hdp_spark-master:/spark/  
docker exec -i -t hdp_spark-master bash  
./spark/bin/spark-shell  
:load /spark/06_customer_demographic.scala  

6. Copy the parquet files from spark container to hive container.  

docker cp hdp_spark-master:/customer_demographics_xml_mined .  
docker cp customer_demographics_xml_mined ra_hive-server:/opt/  
rm -r customer_demographics_xml_mined  

7. Refer 07_customer_demograhics_creation.hql , Creating demographic parquet tables and loading data for performing HIVE analytics.  
 
docker exec -i -t ra_hive-server bash  
hive  
 
8. Refer 08_Hive_analytic_queries.hql , contains advanced analytic queries used on demographics tables (Check it).  

Note :  
        Data Flow ==> Mysql -> Sqoop -> HDFS -> HIVE -> SPARK -> HIVE
