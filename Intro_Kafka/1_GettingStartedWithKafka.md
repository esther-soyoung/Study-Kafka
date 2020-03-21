# Kafka 101: Getting Started
Before starting the installation, there's one thing left that makes our lives easier: *Zookeeper*.  
  
## Zookeeper
Is a *centralized coordination service* that aims to ease managing distributed application, by maintaining configuration information and providing distributed synchronization.  
  
## Installation
### Zookeeper Server
1. Installing Java  
```sh
sudo apt-get install openjdk-8-jdk
```
  
2. Installing Zookeeper  
```sh
wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz  
tar zxf zookeeper-3.4.10.tar.gz  
ln -s zookeeper-3.4.10 zookeeper  
```
  
3. Create `myid` file   
`myid` file resides in the server's data directory, containing a single line of text that indicates the server's id. For example `myid` of server 1 should only contain the text "1".  
Server's data directory is specified by the configuration file parameter **dataDir**.  
* First create data directory.
```sh
mkdir zookeeper_data
```
* Then create `myid` in the data directory.
```sh
echo 1 > ./zookeeper_data/myid
```
  
4. Configuration  
```sh
vi ./zookeeper/conf/zoo.cfg
```
Append following lines to `zoo.cfg`.  
```sh
trickTime=2000
initLimit=10
syncLimit=5
dataDir={path-to-data-directory}
clientPort=2181
server.1={zookeeper-server-ip}:2888:3888
```
  
5. Starting and Stopping Zookeeper
```sh
cd zookeeper
./bin/zkServer.sh start
./bin/zkServer.sh stop
```
  
### Kafka Server
1. Installing Java  
```sh
sudo apt-get install openjdk-8-jdk
```
  
2. Installing Kafka  
```sh
wget http://apache.mirror.cdnetworks.com/kafka/2.4.0/kafka_2.13-2.4.0.tgz
tar zxf kafka_2.13-2.4.0.tgz
ln -s kafka_2.13-2.4.0 kafka
```
  
3. Create data directory 
```sh
mkdir kafka_data
```
  
4. Configuration  
```sh
vi ./kafka/config/server.properties
```
Change following values.
```sh
broker.id=1
log.dirs=~/kafka_data
zookeeper.connect={zookeeper-server-ip}:2181/kafka
```
Append following lines to `server.properties`.  
```sh
delete.topic.enable=true
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://{Kafka-server-ip}:9092
offsets.topic.replication.factor=1
```
  
5. Starting and Stopping Kafka
```sh
cd kafka
./bin/kafka-server-starter.sh ./config/server.properties
./bin/kafka-server-stop.sh
```

## Reference 
[Zookeeper Admin Guide](https://zookeeper.apache.org/doc/r3.3.3/zookeeperAdmin.html)  
[YBIGTA Engineering Team Wiki](https://github.com/YBIGTA/EngineeringTeam/wiki/03.-Kafka-설치)
