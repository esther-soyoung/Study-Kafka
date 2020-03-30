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
The more the Zookeeper servers, easier and faster it gets to handle requests and errors. Scaling out Zookeeper can be done anytime, by bringing more servers in, installing Zookeepers on them, then registering **myid** and configuring **zoo.cfg**.
  
1. Connect to the new Zookeeper server.  
2. Set **myid** by: `echo "{id}" > /data/myid`  
3. Configure **zoo.cfg** by:
    ```sh
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/data
    clientPort=2181
    server.1=esther-zk001:2888:3888
    server.2=esther-zk001:2888:3888
    server.3=esther-zk001:2888:3888
    server.4=esther-zk001:2888:3888
    server.5=esther-zk001:2888:3888
    ```
  
Now the new set Zookeeper servers are good to go. However preexisting servers still need to be updated. Update **zoo.cfg** of these servers, then reboot. Note that it is safe to update the leader at last.  
  
First find out which one is the leader. Connect to Zookeeper servers one by one, execute following Zookeeper command:  
`/usr/local/zookeeper/bin/zkServer.sh status`  
  
Append new lines of newly added ones on **zoo.cfg** of preexisting servers, reboot the servers by:  
`systemctl restart zookeeper-server .service`  
  
Check if the Zookeeper ensemble is working fine(well synchronized). Connect to the leader, execute following command:  
`echo mntr | nc localhost 2181 grep zk_synced_followers`  
This command returns the number of followers being synchronized.
  
## Scaling Out Kafka
Scaling out Kafka is way simpler than that of Zookeeper. Introduce new server, install Kafka on it, set **broker.id** not to be duplicated, then run Kafka!  
Check if the new servers are well joined to the existing Kafka cluster. Connect to any Zookeeper server, run the following command:  
`/usr/local/zookeeper/bin/zkCli.sh`  
This starts a Zookeeper CLI. Then list out the Kafka cluster.  
`ls /esther-kafka/brokers/ids`  
  
Newly added Kafka servers remains idle at first, since no topic and no partition are assigned. Assume there are five partitions in *esther-topic*. 
```sh
/usr/local/kafka/bin/kafka-topics.sh \
--zookeeper esther-zk001:2181,esther-zk002:2181,esther-zk003:2181/esther-kafka \
--topic esther-topic --describe
```
All the partitions are distributed between preexisting Kafka brokers. Partitions need to be manually reassigned.  
  
First create **partition.json** file under `/usr/local/kafka`.
```json
{"version":1,
"partitions":[
    {"topic":"esther-topic","partition":0,"replicas":[2,1]},
    {"topic":"esther-topic","partition":1,"replicas":[3,2]},
    {"topic":"esther-topic","partition":2,"replicas":[4,3]},
    {"topic":"esther-topic","partition":3,"replicas":[5,4]},
    {"topic":"esther-topic","partition":4,"replicas":[1,5]}
]}
```
  
Run following command:  
```sh
/usr/local/kafka/bin/kafka-reassign-partitions.sh \
--zookeeper esther-zk001:2181,esther-zk002:2181,esther-zk003:2181\esther-kafka \
--reassignment-json-file /usr/local/kafka/partition.json --execute
```
  
## Monitoring Kafka

