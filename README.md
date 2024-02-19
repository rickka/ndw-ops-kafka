## Add / Register a postgresql Source Connector:
### Run
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
    "table.include.list": "public.core_brand,public.core_cooler,public.core_coolermanufacturer,
                          public.core_coolermodel,public.core_distributor,public.core_district,
                          public.core_outlet,public.core_technician",
    "schema.history.internal.kafka.bootstrap.servers": "kafka:9092",
    "schema.history.internal.kafka.topic": "schema-changes.bdi_dev",
    "plugin.name": "pgoutput",
    "topic.prefix": "ndw_"
 }
}'


## Add / Register a postgresql Sink Connector:
curl -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d \
'{
  "name": "pg-sink-connector",
  "config": {
    "heartbeat.interval.ms": "3000",                                                            
    "autoReconnect":"true",
    "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
    "tasks.max": "3",
    "connection.url": "jdbc:postgresql://remote_host:5432/database_name",
    "connection.user": "user",
    "connection.password": "password",
    #"topics": "my_topic",
    "auto.create": "true",
    "auto.evolve": "true",
    "insert.mode": "upsert",
    "delete.enabled": "true",
    "topics":"ndw_.public.core_brand,ndw_.public.core_cooler,ndw_.public.core_coolermanufacturer,
              ndw_.public.core_coolermodel,ndw_.public.core_distributor,ndw_.public.core_district,
              ndw_.public.core_outlet,ndw_.public.core_technician"
  }
}''


<!-- {
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


## View connectors
### Run
curl -i -X GET localhost:8083/connectors/


## Check connectors
### Run
curl -s http://localhost:8083/connectors/pg-DB-connector/status
#This should return a JSON response with the connector status.


## Remove connector
### Run
curl -i -X DELETE localhost:8083/connectors/<connector-name>/
i.e curl -i -X DELETE localhost:8083/connectors/pg-DB-connector/


## To Modify connector
curl -i -X PUT -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/inventory-connector/config -d '{ "connector.class": "io.debezium.connector.mysql.MySqlConnector", "tasks.max": "1", "database.hostname": "mysql", "database.port": "3306", "database.user": "debezium", "database.password": "dbz", "database.server.id": "184054", "database.server.name": "dbserver1", "database.include.list": "inventory", "database.history.kafka.bootstrap.servers": "kafka:9092", "database.history.kafka.topic": "dbhistory.inventory" }'


# MONITORING
To check if Debezium (which captures database changes and publishes them to Kafka) has successfully read your database, you can use Kafka tools to inspect the topics where Debezium publishes the change events. Debezium typically publishes change events to Kafka topics, and you can monitor these topics to see if any data is being produced.


## 1. List Kafka Topics 
If Not Dockerized:
    kafka-topics.sh --list --bootstrap-server localhost:9092
If Dockerized:
    docker exec -it <kafka_container_id> /bin/bash #To get into the container terminal

    In the container terminal, run:
        kafka-topics.sh --list --bootstrap-server 127.0.0.1:9092


## 2. Inspect Debezium Topics
If Not Dockerized:
    kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my_database.my_table --from-beginning
if Dockerized:
    docker exec -it <kafka_container_id> kafka-console-consumer.sh --bootstrap-server <kafka_container_name>:9092 --topic my_database.my_table --from-beginning

## 3.Check for Data: If Debezium is successfully reading your database and publishing change events to Kafka, you should see data being printed to the console when you consume messages from the relevant topics. If you don't see any data, there may be an issue with your Debezium configuration or connectivity to the database.



View Data in Topics:

Once you know the topics you want to consume from, you can use the kafka-console-consumer.sh script to view the data. For example, to consume messages from a topic named my_topic, you can use the following command:
docker exec -it <kafka_container_id> kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic ndw_.public.core_brand --from-beginning
