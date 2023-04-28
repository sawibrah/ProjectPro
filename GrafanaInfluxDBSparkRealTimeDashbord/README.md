# Build a real-time dashboard with spark, grafana and InfluxDB

Use Spark , Grafana, and InfluxDB to build a real-time e-commerce users analytics dashboard by consuming different events such as user clicks, orders, demographics

# Tech Stack:
Language: Java8, SQL
Services: Kafka, Spark Streaming, MySQL, InfluxDB, Grafana, Docker, Maven

# Approach:
User purchase events in Avro format are produced via Kafka.

Spark Streaming Framework does join operations on batch and real-time events of user purchase and demographic type.

MySql Holds the demographic data such as age, gender, country, etc.

Spark Streaming Framework consumes these events and generates a variety of points suitable for time series and dashboarding.

Kafka connect pushes the events from the Kafka streams to influxDB.

Grafana connects to different sources like influxDB, MySQL and populates the graphs.

# High-level design

![alt text](https://github.com/sawibrah/ProjectPro/blob/master//GrafanaInfluxDBSparkRealTimeDashbord/img/highleveldesign.png?raw=true)
