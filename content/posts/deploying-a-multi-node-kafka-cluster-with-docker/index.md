---
title: "Deploy a multi-node, multi-server Kafka Cluster with Docker"
date: 2017-12-15
description: "A Docker deployment of Kafka avoids the need to manually configure each broker and provides a very simple and scalable installation methodology; particularly over multiple servers."
#cover: banner.png
tags: [guides, docker, kafka, networking]
comments: false
toc: false
---

## What is Kafka?
[Essentially:][kafka_intro]
> Kafka is an open-source, very scalable, distributed messaging platform by Apache. It is designed to handle large volumes of data in real-time efficiently.

Kafka works on the concept of a "publish-subscribe" methodology:

- "Producers" will push content to the Kafka cluster, to a destination "topic".
- A Kafka cluster is managed by Zookeeper, and can contain one or more "Brokers".
- Brokers manage topics and the associated ingress and egress messages.
- "Consumers" subscribe to specific topics and will receive a continuous stream of data from the cluster for as long as there is a Producer pushing content to that topic.

Topics have two key properties: _partitions_ and _replication factors_.
   - _Partitions_ logically separate the messages within a topic into small "groups" which are distributed between all available brokers. For example, if there are 3 brokers in a cluster, and a topic has 6 partitions, Broker #1 might be assigned Partition #3 & #6, Broker #2 - Partition #1 & #5, and Broker #3 - Partition #2 & #4.
   - A _replication factor_ helps add redundancy to a topic and its partitions across the available Brokers. Using the example above, if the same topic has a replication factor of 3, then each broker would have a copy of the partitions managed by other brokers in addition to its own. Hence, if a broker goes down, the messages stored in the partitions held by it are not lost.

The cool thing about Kafka consumers is that, when they are grouped together and used in conjunction with partitions, they can automatically load balance incoming messages between them!

This is really handy for avoiding Consumer bottlenecks if you are dealing with real-time data in high volumes. If a Consumer cannot process the incoming messages fast enough, it will fall behind and you will begin to notice an increasing delay between the current time and the incoming message. If a second Consumer is added to the same group as the first, and both are subscribed to a topic with 6 partitions, then Consumer #1 might be assigned messages from Partition #1, #2, #5, and Consumer #2 might be assigned messages from Partition #3, #4, #6. This is instead of Consumer #1 bottlenecking by having to process Partitions #1-6 by itself.

## Why Docker?
Deploying Kafka in Docker greatly simplifies deployment as we do not need to manually configure each broker individually!

We can use single Docker Compose file to deploy Kafka to multiple server instances using Docker Swarm in a single command. Additionally, Docker allows us to easily scale up our cluster to additional nodes in the future if we require.

## Requirements
When deployed via Docker, Kafka tends to use approximately 1.3 GB - 1.5 GB of RAM per broker - so make sure your server instances have enough memory allocated and available.

## Process
Before proceeding, make sure you have Docker installed on ALL of the servers you wish to deploy Kafka to.

### Part 1 - Create a Docker Swarm
To begin, we need to initialize a Docker Swarm. A Swarm will consist of at least one Master Node, and one or more Worker Nodes (or, if you just have a single server instance, a single Master Node).

#### 1. Initialize the Swarm & Master Node
On the server instance you have designated to be the Master, initialize the Swarm. This will set the current instance as the Master node:

```shell
docker swarm init --advertise-addr [MANAGER-IP]:2377
```
Where `[MANAGER-IP]` is the static IP address of the server instance.
You can find this using `ifconfig`.

This will generate a unique command which you will run on all other nodes in the next section in order to designate them as Worker nodes.

Save this command in a safe place.

Example (__your command and token will be different!__):

```shell
docker swarm join --token SWMT[...]ewrp [MANAGER-IP]:2377
```

#### 2. Check your firewall
Swarm nodes use port 2377 to communicate with the Master node which must not be blocked by any firewall.

Additionally, Swarm daemons use ports 7946 (tcp/udp) and 4789 (udp) to communicate.

Make sure your all of your instances are able to communicate:
- TCP/UDP traffic on ports 2377 and 7946.
- UDP traffic on port 4789.

#### 3. Configure the Worker Nodes
On all other server instances you wish to deploy Kafka on, run the command generated in the step #1 above. This will add these instances to the Docker Swarm as Worker Nodes.

If you are having issues:
- As per Step #2 above, make sure all your instances can communicate on the specified ports.
- Make sure all of your instances have static IP addresses assigned to them.

#### 4. Verify the Swarm
On the __Master Node__ only, run the following command to check the status of all nodes in the swarm:

```shell
docker node ls
```


### Part 2 - Deploying Kafka
#### 1. Create docker-compose.yml
Copy the following into a file named `docker-compose.yml` on the Master Node:

```yaml
version: '3.2'
services:
   zookeeper:
      image: wurstmeister/zookeeper
      ports:
         - "2181:2181"

   kafka:
      image: wurstmeister/kafka:latest
      deploy:
         mode: global
      ports:
         - target: 9094
           published: 9094
           protocol: tcp
           mode: host
      environment:
         HOSTNAME_COMMAND: "docker info | grep ^Name: | cut -d' ' -f 2" # Normal instances
         # HOSTNAME_COMMAND: "curl http://169.254.169.254/latest/meta-data/public-hostname" # AWS Only
         KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
         KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
         KAFKA_ADVERTISED_PROTOCOL_NAME: OUTSIDE
         KAFKA_ADVERTISED_PORT: 9094
         KAFKA_PROTOCOL_NAME: INSIDE
         KAFKA_PORT: 9092
         KAFKA_CREATE_TOPICS: myTopic:3:3,anotherTopic:2:2
      volumes:
         - /var/run/docker.sock:/var/run/docker.sock
```
- The Zookeeper container is responsible for managing the Kafka cluster.
- Using the above configuration, you will be able to connect to your Kafka brokers using port 9094 at their designated IP/hostname.
- Alter `CREATE_KAFKA_TOPICS` in to pre-allocate your topic names. Separate multiple topics with commas.
    - In the file above, two topics will be created: One with 3 partitions and 3 replications, and another with 2 partitions and 2 replications.
- You can specify an `environment` entry for most (if not all) Kafka configuration options. For more information, see [here](https://github.com/wurstmeister/kafka-docker#pre-requisites).

<br>__If you are deploying to AWS!__<br>
Comment out the __first__ `HOSTNAME_COMMAND` line under `environment` and uncomment the __second__ `HOSTNAME_COMMAND`.

There is a different method to resolving the instance hostname for AWS instances, which is critical to Kafka. __Failing to do this will mean that while your broker(s) will be deployed, nothing will be able to find or connect to them.__

#### 2. Check your firewall (again)
- Kafka communicates with Zookeeper on port 2181.
- Kafka will listen for connections on 9094 and communicate internally on port 9092.
- Make sure these ports are open for TCP/UDP traffic or you may have communication issues.

#### 3. Deploy the Kafka stack
On the Master Node, run the following command:

```shell
docker stack deploy --compose-file docker-compose.yml kafka
```

- This will deploy a Kafka broker to each node in the Swarm, and bring online ONE Zookeeper management container.
- Additionally, any Kafka topics specified in the `docker-compose.yml` file will be initialized.
- The deployed service will have the name `kafka`.

#### 4. Verify status
You can use the following command to verify the status of the Kafka stack:

```shell
docker stack services kafka
```

The status for all containers should be shown. A successful launch on 3 servers (for example) should show 3/3 replicas for Kafka and 1/1 replicas for Zookeeper.

### Part 3 - Stopping the cluster
Simply run the following command on the Master node:

```shell
docker stack rm kafka
```

### Part 4 - Sending & Receiving data using your Kafka cluster
The Kafka stack deployed above will initialize a single Kafka container on each node within the Swarm. Hence the IP address of each node is the IP address of a Kafka broker within the Kafka cluster.

The Kafka brokers will listen for Consumer applications and Producers on port 9094.

It does not matter which broker IP/hostname you use for the producer/consumer connection.
In many applications or APIs (eg: Telegraf, kafka-python etc) in which you can specify a list of brokers, only the first broker is used. The others are used as fallback (in the specified order) in case the preceding broker is unavailable or down.


[kafka_intro]: https://kafka.apache.org/intro.html
