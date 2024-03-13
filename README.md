# Setup
Assumes a base environment such as:

**Linux / Ubuntu 22.04** running on your machine

**Source PostgreSQL Database** # To pull data from

**Sink PostgreSQL DataBase** # To stream data into

**Docker / Docker Compose** # To manage 3 instances for **Kafka**, **Kafka-Connect** and **Zookeeper**

<br />

To start the docker instances:
#### **Run** (from within the same directory as the **docker-compose.yml** file)
<pre>
docker compose up 
</pre>
Initial run should take afew minutes, because the docker images need to be downloaded.
<br />
<br />

# Initialization
## To add / register a postgresql Source Connector:
#### **Run**
<pre>
curl -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d \
'{
 "name": "pg-source-connector",
 "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "tasks.max": "1",
    "database.hostname": "127.0.0.1",
    "database.port": "5432",
    "database.user": "your_user",
    "database.password": "your_password",
    "database.dbname" : "your_db",
    "database.whitelist": "your_db",
    "table.include.list": "public.table_1,public.table_2,public.table_3, public.table_4"
    "schema.history.internal.kafka.bootstrap.servers": "kafka:9092",
    "schema.history.internal.kafka.topic": "schema-changes.bdi_dev",
    "plugin.name": "pgoutput",
    "topic.prefix": "ndw_"
 }
}'
</pre>
<br />
<br />

## To add / register a postgresql Sink Connector:
#### **Run**
<pre>
curl -X POST -H "Accept:application/json" -H "Content-Type:application/json" http://localhost:8083/connectors/ -d \
'{
  "name": "pg-sink-connector",
  "config": {
    "heartbeat.interval.ms": "3000",                                                            
    "autoReconnect":"true",
    "connector.class": "io.debezium.connector.jdbc.JdbcSinkConnector",
    "tasks.max": "3",
    "connection.url": "jdbc:postgresql://127.0.0.1:5433/your_staging_or_sink_db",
    "connection.username": "staging_db_username",
    "connection.password": "staging_db_password",
    "auto.create": "true",
    "auto.evolve": "true",
    "insert.mode": "upsert",
    "primary.key.mode": "record_key",
    "primary.key.fields": "id",
    "delete.enabled": "true",
    "table.include.list": "public.table_1,public.table_2,public.table_3,public.table_4",
    "topics":"ndw_.public.table_1,ndw_.public.table_2,ndw_.public.table_3,ndw_.public.table_4"
  }
}'
</pre>
<br />
<br />
<!--
#"topics": "my_topic", {
"name": "mysql-sink-connector",  
"config": {
    "heartbeat.interval.ms": "3000",                                                            
    "autoReconnect":"true",
    "connector.class": "io.debezium.connector.jdbc.JdbcSinkConnector",  
    "tasks.max": "3",  
    "connection.url":"jdbc:mysql://10.10.10.10:3306/mydatabase_sink",   # specify sink DB
    "connection.username": "root",  
    "connection.password": "*********",  
    "insert.mode": "upsert",  
    "delete.enabled": "true",  
    "primary.key.mode": "record_key",  
    "schema.evolution": "basic",  
    "database.time_zone": "UTC",
    "auto.evolve": "true",
    "quote.identifiers":"true",
    "auto.create":"true",                                 # auto create tables
    "value.converter.schemas.enable":"true",              # auto reflect schema changes
    "value.converter":"org.apache.kafka.connect.json.JsonConverter",
    "table.name.format": "${topic}",
    "topics.regex":"exampleserver_source.mydatabase_source.*",  # topics regexp to replicate
    "pk.mode" :"kafka"
  }
} -->


## To view the connectors you have registered.
#### **Run**
<pre>
curl -i -X GET http://localhost:8083/connectors/
</pre>
<br />
<br />

## To check the status of a connector
#### **Run**
<pre>
curl -s http://localhost:8083/connectors/pg-DB-connector/status
</pre>
#### # this should return a JSON response with the connector status.
<br />
<br />

## To Remove a connector
#### **Run**
<pre>
curl -i -X DELETE localhost:8083/connectors/your_connector_name/
</pre>
i.e curl -i -X DELETE localhost:8083/connectors/pg-DB-connector/
<br />
<br />

## To Modify a connector
#### **Run**
<pre>
curl -i -X PUT -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/inventory-connector/config -d 
  '{ "connector.class": "io.debezium.connector.mysql.MySqlConnector", 
      "tasks.max": "1", 
      "database.hostname": "mysql", 
      "database.port": "3306", 
      "database.user": "debezium", 
      "database.password": "dbz", 
      "database.server.id": "184054", 
      "database.server.name": "dbserver1", 
      "database.include.list": "inventory", 
      "database.history.kafka.bootstrap.servers": 
      "kafka:9092", 
      "database.history.kafka.topic": "dbhistory.inventory"
  }'
</pre>
<br />
<br />
<br />

# Monitoring
To check if Debezium (which captures database changes and publishes them to Kafka) has successfully read your database, you can use Kafka tools to inspect the topics where Debezium publishes the change events. Debezium typically publishes change events to Kafka topics, and you can monitor these topics to see if any data is coming in.
<br />
<br />

## 1. List Kafka Topics 
If Not Dockerized:
  <pre>
  kafka-topics.sh --list --bootstrap-server localhost:9092
  </pre>
If Dockerized, first connect to the container bash terminal:
  <pre>
  docker exec -it <kafka_container_id> /bin/bash
  </pre>
  then **run**:
  <pre>
  kafka-topics.sh --list --bootstrap-server 127.0.0.1:9092
  </pre>


<br />
<br />

## 2. Inspect Debezium Topics
If Not Dockerized:
  <pre>
  kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic your_database.your_table --from-beginning
  </pre>
If Dockerized:
  <pre>
  docker exec -it your_kafka_container_id kafka-console-consumer.sh --bootstrap-server your_kafka_container_name:9092 --topic your_topic.your_table --from-beginning
  </pre>
<br />
<br />

## 3. Check for Data: 
If Debezium is successfully reading your database and publishing change events to Kafka, you should see data being printed to the console when you consume messages from the relevant topics. If you don't see any data, there may be an issue with your Debezium configuration or connectivity to the database.
<br />
<br />


### **View Data in Topics:**

Once you know the topics you want to consume from, you can use the kafka-console-consumer.sh script to view the data. For example, to consume messages from a topic named my_topic, you can use the following command:

If Dockerized:
<pre>
docker exec -it your_kafka_container_id kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic your_topic.your_table --from-beginning
</pre>
