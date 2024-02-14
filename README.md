## Add / Register a postgresql connector:
### Run
curl -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d \
'{
 "name": "pg-DB-connector",
 "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "tasks.max": "1",
    "database.hostname": "127.0.0.1",
    "database.port": "5432",
    "database.user": "postgres",
    "database.password": "creation",
    "database.dbname" : "bdi_dev",
    "database.whitelist": "bdi_dev",
    "schema.history.internal.kafka.bootstrap.servers": "kafka:9092",
    "schema.history.internal.kafka.topic": "schema-changes.bdi_dev",
    "plugin.name": "pgoutput",
    "topic.prefix": "ndw_"
 }
}'


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