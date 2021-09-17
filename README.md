![image](../images/confluent-logo-300-2.png)

# Overview

This example shows how KSQL can be used to process a stream of click data, aggregate and filter it, and join to information about the users.
Visualisation of the results is provided by Grafana, on top of data streamed to Elasticsearch. 

# Documentation

You can find the documentation for running this example and its accompanying tutorial at [https://docs.confluent.io/platform/current/tutorials/examples/clickstream/docs/index.html](https://docs.confluent.io/platform/current/tutorials/examples/clickstream/docs/index.html?utm_source=github&utm_medium=demo&utm_campaign=ch.examples_type.community_content.clickstream)

# Download

You need to download datagen and elasticsearch jars

```
docker run -v $PWD/confluent-hub-components:/share/confluent-hub-components confluentinc/ksqldb-server:0.8.0 confluent-hub install --no-prompt confluentinc/kafka-connect-datagen:0.4.0
docker run -v $PWD/confluent-hub-components:/share/confluent-hub-components confluentinc/ksqldb-server:0.8.0 confluent-hub install --no-prompt confluentinc/kafka-connect-elasticsearch:10.0.2
```

# Docker compose

Then you need to build docker container 
```
docker-compose up -d
```

After it we can use this command to ensure that everything is ok

```
docker-compose ps
```

![img.png](screenshots/img.png)

# Create clickstream data

Launch ksqlDB CLI by this command:
```
docker-compose exec ksqldb-cli ksql http://ksqldb-server:8088
```
![img_1.png](screenshots/img_1.png)

Then run this script to create three source connectors and generate mock data

```
RUN SCRIPT '/scripts/create-connectors.sql';
```
Output will be similar 

![img_2.png](screenshots/img_2.png)

Now the clickstream generator is running, simulating the stream of clicks. Sample the messages in the clickstream topic

```
print clickstream limit 3;
```

![img_3.png](screenshots/img_3.png)

The second data generator running is for the HTTP status codes. Sample the messages in the clickstream_codes topic

```
print clickstream_codes limit 3;
```

![img_4.png](screenshots/img_4.png)

The third data generator is for the user information. Sample the messages in the clickstream_users topic

```
print clickstream_users limit 3;
```

![img_5.png](screenshots/img_5.png)

Go to Confluent Control Center UI at http://localhost:9021 and view the three kafka-connect-datagen source connectors created with the ksqlDB CLI.

![img_6.png](screenshots/img_6.png)

# Load the Streaming Data to ksqlDB

Load the statements.sql file that runs the tutorial app

```
RUN SCRIPT '/scripts/statements.sql';
```

![img_7.png](screenshots/img_7.png)

![img_8.png](screenshots/img_8.png)

![img_9.png](screenshots/img_9.png)

![img_10.png](screenshots/img_10.png)

# Verify the data

![img_11.png](screenshots/img_11.png)

Verify that data is being streamed through various tables and streams. Query one of the streams CLICKSTREAM:

![img_12.png](screenshots/img_12.png)
# Load the Clickstream Data in Grafana

Set up the required Elasticsearch document mapping template

```
docker-compose exec elasticsearch bash -c '/scripts/elastic-dynamic-template.sh'
```

Run this command to send the ksqlDB tables to Elasticsearch and Grafana:

```
docker-compose exec ksqldb-server bash -c '/scripts/ksql-tables-to-grafana.sh'
```

![img_13.png](screenshots/img_13.png)

Load the dashboard into Grafana.

```
docker-compose exec grafana bash -c '/scripts/clickstream-analysis-dashboard.sh'
```

Navigate to the Grafana dashboard at http://localhost:3000. Enter the username and password as user and user. Then navigate to the Clickstream Analysis Dashboard.

![img_14.png](screenshots/img_14.png)

In the Confluent Control Center UI at http://localhost:9021, again view the running connectors. The three kafka-connect-datagen source connectors were created with the ksqlDB CLI, and the seven Elasticsearch sink connectors were created with the ksqlDB REST API.

![img_15.png](screenshots/img_15.png)

