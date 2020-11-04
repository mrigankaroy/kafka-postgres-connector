# kafka-postgres-connector
Kafka Postgres source connector example - Docker, Kafka, Postgres and Debezium source connector

This document will help to setup Postgres, Apache Kafka, Debezium mongodb source connector on Docker

## 1. Create docker network

[Ref](https://github.com/mrigankaroy/kafka-mongo-connector#1-create-docker-network)

## 2. Postgres Database Setup

Create a postgresql volume for store data

```sh
$ docker create -v /var/lib/postgresql/data --name PostgresData --network=library-network alpine
```

Create postgres db image

```sh
$ docker run -p 5432:5432 --name postgres -e POSTGRES_PASSWORD=admin -d --volumes-from PostgresData --network=library-network postgres
```

Open pgAdmin and execute SQL

```sh
ALTER SYSTEM SET wal_level = logical;
```

Restart postgres instance.

## 3. Kafka Setup

[Ref](https://github.com/mrigankaroy/kafka-mongo-connector#3-kafka-setup)

## 4. Debezium source connector Setup

[Ref](https://github.com/mrigankaroy/kafka-mongo-connector#4-debezium-source-connector-setup)

### 5. Testing

Open pgAdmin and create new database test_database

Create table inside test_database

``` sql
CREATE TABLE test_connector (
    id serial PRIMARY KEY,
    text varchar NOT NULL
);
```
Create new connector 

``` sh
curl --location --request POST 'http://localhost:8083/connectors' \
--header 'Content-Type: application/json' \
--data-raw '{
  "name": "test_inventory-connector",  
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector", 
    "database.hostname": "postgres", 
    "database.port": "5432", 
    "database.user": "postgres", 
    "database.password": "admin", 
    "database.dbname" : "testDb", 
    "database.server.name": "test_database", 
    "table.include.list": "public.test_connector" ,
    "slot.name" : "public_test_connector_slot",
    "plugin.name": "pgoutput"

  }
}'
```

Check newly created topic in kafka

```sh
$ bin/kafka-topics.sh --list --zookeeper zookeeper:2181

```

Monitor messages in kafka topic

```sh
$ bin/kafka-console-consumer.sh --topic testDb.public.test_connector --from-beginning --bootstrap-server kafka:9092

```
