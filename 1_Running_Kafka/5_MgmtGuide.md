# Kafka Management Guide
As a large scale distributed system, Kafka is not an easy application to manage. There is a need to get familiar with Kafka management applications such as Zookeeper, JMX or Kafka Manager.
  
## Basic Kafka Commands
Basic Kafka commands can be found in Kafka `bin/` directory(i.e. `/usr/local/kafka/bin/`).
  
### Creating Topic
Topics can be created with `kafka-topics` command.  
```sh
/usr/local/kafka/bin/kafka-topics.sh \
--zookeeper esther-zk001:2181,esther-zk002:2181,esther-zk003:2181/esther-kafka \
--replication-factor 1 --partitions 1 --topic esther-topic --create
```
  
### List of Topics
`--list` option lists out currently exising topics.  
```sh
/usr/loca/kafka/bin/kafka-topics.sh \
--zookeeper esther-zk001:2181,esther-zk002:2181,esther-zk003:2181/esther-kafka \
--list
```
  
### Topic Details
`--describe` option gets a detailed information for each topic.  
```sh
/usr/loca/kafka/bin/kafka-topics.sh \
--zookeeper esther-zk001:2181,esther-zk002:2181,esther-zk003:2181/esther-kafka \
--topic esther-topi --describe
```
  
### Changing Settings
Configuration can be changed by `kafka-config` command.  
```sh
/usr/loca/kafka/bin/kafka-topics.sh \
--zookeeper esther-zk001:2181,esther-zk002:2181,esther-zk003:2181/esther-kafka \
--alter --entity-type topics --entity-name esther-topic --add-config retention.ms=3600000
```  
```sh
/usr/loca/kafka/bin/kafka-topics.sh \
--zookeeper esther-zk001:2181,esther-zk002:2181,esther-zk003:2181/esther-kafka \
--alter --entity-type topics --entity-name esther-topic --delete-config retention.ms=3600000
```
  
### Creating More Partitions 
Note that only increasing the number of paritions is allowed, decreasing is not allowed. Along with partitions, the number of *consumers* should also be increased in order to improve the efficiency.  
```sh
/usr/loca/kafka/bin/kafka-topics.sh \
--zookeeper esther-zk001:2181,esther-zk002:2181,esther-zk003:2181/esther-kafka \
--alter --topic esther-topic --partitions {target_num}
```
  
### Adjusting Replication Factor
First create `.json` file with the locations of partitions.  
```json
{"version":1,
"partitions":[
    {"topic":"esther-topic","partition":0,"replicas":[1,2]},
    {"topic":"esther-topic","partition":1,"replicas":[2,3]}
]}
```  
Partition 0 has two replicas, the leader is in broker1, the follower is in broker2.  
Partition 1 has two replicas, the leader is in broker2, the follower is in broker3.
  
To increase the replication factor, simply append a broker to the list of *replicas*, vice versa.
  
Now execute the updated json file with `--reassignment-json-file`.  
```sh
/usr/loca/kafka/bin/kafka-reassign-partitions.sh \
--zookeeper esther-zk001:2181,esther-zk002:2181,esther-zk003:2181/esther-kafka \
--reassignment-json-file {path_to_json_file} --execute
```
  
### List out Consumer Group
List of consumers can be found by `kafka-consumer-groups` command with `--bootstrap-server` option.  
```sh
/usr/loca/kafka/bin/kafka-consumer-groups.sh \
--bootstrap-server esther-kafka001:9092,esther-kafka002:9092,esther-kafka003:9092 \
--list
```
  
### Consumer Status and Offset
`kafka-consumer-group` also provides detailed status of consumers.  
```sh
/usr/loca/kafka/bin/kafka-consumer-groups.sh \
--bootstrap-server esther-kafka001:9092,esther-kafka002:9092,esther-kafka003:9092 \
--group esther-consumer --describe
```
  
## Scaling Out Zookeeper
