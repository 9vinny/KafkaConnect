#KafkaConnect 
#(Kafka FileStream Connectors)
### --Use Kafka Connect to import/export data

## Introduction

Writing data from the console and writing it back to the console is a convenient place to start, but you'll probably want to use data from other sources or export data from Kafka to other systems. For many systems, instead of writing custom integration code you can use Kafka Connect to import or export data. It can be used for moving data in and out of commonly used systems such as databases, HDFS, and Elasticsearch.

Kafka Connect is a framework that provides scalable and reliable streaming of data to and from Apache Kafka. With Kafka Connect, writing a topic’s content to a local text file requires only a few simple steps

## Lab Configuration

In this lab, we'll start two connectors running in standalone mode: we will use the **FileStreamSink** connector class for piping the content of a file to producer(file source), and directing file content to another destination(file sink).

### Three configuration files:
**connect-standalone.properties**: 
configuration for the Kafka Connect process, containing common configuration such as the Kafka brokers to connect to and the serialization format for data.

**connect-file-source.properties**: includes unique connector name, the connector class to instantiate, and any other configuration required for file source.

**connect-file-sink.properties**: includes unique connector name, the connector class to instantiate, and any other configuration required for file sink.

## Lab Steps
(1) Start Kafka and Zookeeper Server (if you haven't yet)

1. `cd /opt/kafka_2.12-1.1.0/`  

2. `bin/kafka-server-start.sh config/server.properties`

(2) Create a new topic in a **NEW** Terminal

1. `cd /opt/kafka_2.12-1.1.0/`  

2. `bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic connect-test`

(3) Create some seed data on **NEW** Terminal

1. `cd /opt/kafka_2.12-1.1.0/`  

2. `echo -e "foo\nbar" > test.txt`  

3. It is very important to make sure we create the file under this directory (not any sub-directories) to accommadate the default setting in the connectors' configuration, unless you otherwise change the config files.

(4) Always check to see whether Kafka Connect process is still running in the background.

Kafka Connect Kafka Connect is intended to be run as a service, by default KafkaConnect Worker runs on port `8083`. And `Ctrl + Z` might not be able to kill it succesfully in the background. Same as `kill` or `pkill` command. 

1. `cd /opt/kafka_2.12-1.1.0/`  

2. `netstat -tulpn | grep 8083`  

3. `kill -9 <PID from grep>`  

4. run `netstat -tulpn | grep 8083` again to make sure previous KafkaConnect Worker process is killed.

5. [Kill background worker process](Images/kill_worker_in_background.png)

(5) In the same Terminal as above, run the KafkaConnect process

1. `bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties`

2. You'll see a number of log messages, including some indicating that the connectors are being instantiated. 

3. Then, the source connector should start reading lines from `test.txt` and producing them to the topic `connect-test`.

4. And the sink connector should start reading messages from the topic `connect-test` and write them to the file `test.sink.txt`.

(6) Verify whether the data has been delivered through the pipeline by examining the output file.

1. `cd /opt/kafka_2.12-1.1.0/`  

2. `more test.sink.txt`

(7) Data is being stored in the Kafka topic `connect-test`, so we can also run a console consumer to see the data in the topic

1. `bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning`


## Assignment
(1) Put the `big.data.syllabus.txt` file into the main directory under Kafka  

(2) create a new topic called `big-data`

(3) Change the configurations for `connect-file-source.properties` and `connect-sink-source.properties` to make sure the source file name and sink file name (please name it `sink.syllabus.txt`)  are correct, topic is correct.

(4) Run the FileStreaming process in standalone mode as in the lab

(5) Make a screenshot to verify that `sink.syllabus.txt` is under your main directory and get content delivered as shown in the example below:  
[View from CLI](Images/view_from_CLI.png)

(6) After running the consumer, make a screenshot of the consumed messages in json format:  
[View from consumer](Images/view_from_consumer.png)


Note: since both source and sink connectors can track offsets, so if you already consume the same file with the same content using these connectors once, then the next time you run it, the same content won't be flushed again. If somehow you want to re-run the process for same content, please recreate the docker image by running `docker-compose up --force-recreate`

## References:
https://kafka.apache.org/quickstart#quickstart_kafkaconnect 
https://docs.confluent.io/current/connect/quickstart.html  
https://docs.confluent.io/current/connect/connect-filestream/filestream_connector.html  
http://bigdatums.net/2017/06/22/writing-data-from-apache-kafka-to-text-file/







