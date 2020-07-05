# 搭建集群
采用docker-compose 编排服务，搭建Hadoop Spark 分布式计算集群

集群为2个物理节点 每个节点256G内存，10T存储空间，分别名为``bigdata01``、``bigdata02``  

两个节点间通过**Swarm** 通信

## Bigdata01
    从节点，主要是DataNode 和 Worker

 - Hadoop服务
   - DataNode
   - NodeManager
 - Spark服务
   - SparkWorker

## Bigdata02
    主节点
 - Hadoop服务
   - NameNode
   - DataNode
   - ResourceManager
   - NodeManager
 - Spark服务
   - SparkMaster
   - SparkWorker
   - spark-notebook
 - Hive服务
   - hive-metastore-postgresql
   - hive-metastore
   - hive-server
 - Hue

##  启动服务步骤

### 启动docker
 
 ```shell
 systemctl start docker
 ```
### 搭建Swarm网络

bigdata02: 启动swarm manager

```shell
(bigdata02) docker swarm init
```

bigdata01： 加入swarm
```shell
(bigdata01) docker swarm join --token 'your_token' 'your_ip_address:2377'
```

bigdata02: 创建overlay网络
```shell
(bigdata02) docker network create --driver overlay --attachable hadoop_spark_net --subnet 172.18.0.0/25
```

bigdata02: 起个服务真正打通overlay网络
```shell
(bigdata02) docker service create  --replicas 2  --name test-ovs --network hadoop_spark_net cirros
```
查看下网络内部结构
```shell
(bigdata02) docker network inspect hadoop_spark_net(your network name)
```

### 开始docker-compose编排
 - bigdata01
   - 开启服务
        ```
        docker-compose -f docker-hadoop-spark-hive-compose.yml up -d
        ```
   - 关闭服务：
        ```
        docker-compose -f docker-hadoop-spark-hive-compose.yml down
        ```
 - bigdata02
   - 开启服务
        ```
        docker-compose -f docker-hadoop-spark-hive-compose.yml up -d
        ```
   - 关闭服务：
        ```
        docker-compose -f docker-hadoop-spark-hive-compose.yml down
        ```