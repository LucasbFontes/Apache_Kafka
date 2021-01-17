## Context

By November 2020 I enrolled in a [Data Engineer Formation](https://www.datascienceacademy.com.br/bundles?bundle_id=formacao-engenheiro-de-dados) at Data Science Academy, this formation gather 5 courses: DataWarehouse Implementation; DataLake - Design Integration and Integration; High Disponibility and Security; Machine Learning and AI in Distributed Systems and Analytics with Azure. Currently I'm in the DataLake course(which is the #2 of the formation). 

During this course I will have contact with many Big Data tools(such as, kafka, NiFi, streamsets, Flume, SQOOP, among others). As a matter of fact, thanks to this formation I could built my own DataLake using Hadoop as File Store System and host it in AWS using EC2(Elastic Cloud Computing), my Datalake has 1 NameNode and 2 DataNodes.

The project that I'll show now it's one that the instructor did on this course and I replied on my cluster.

## Technologies & Tools

To be able to complete this project I've used the following:

* Hadoop 3.1.0
* AWS EC2
* 1 Namenode with 4gb RAM
* 2 Datanode with 2gb RAM
* [Apache Kafka](https://kafka.apache.org/) - 2-11-1.1.0
* [Streamsets](https://streamsets.com/)
* [Apache Zookeeper](https://zookeeper.apache.org/)

## First..a little theory

Apache Kafka is a message system that has the goal to make data transfer between applications(spoiler: this is what we're going to do here). It works basically with three components:

* Producer: Responsible to produce the data, which means collect the data within the source and transfers it to the broker
* Broker: It's where the data is stored before gets consumed.
* Consumer: Reponsible to consume the data from the broker and connect it into another source

You might be asking yourself why do we need a tool that consumes data, store it, and then transfer it to other sources when I could only transfer the data between these two sources. And the answer is..reliability(among others of course). Let's say you're transfering information without Kafka and somehow you lost your internet connection. What will happened? Well, it's highly possible that your lost your work and have to transfer the data again. However with Kafka, since all the data comes first to the broker if your lost internet connection your work is saved into it and all you have to do it's reconnect and the data will begin to transfer.

The broker stores the data like a queue which means that every data is sent at a time(the first one to enters the queue is the first one to leave) but this happens at a very high speed so it looks almost instantly. Alright if my internet fails the broker can handle it, but what if the broker fails? Well that's why we have a **multi broker**. The multi broker works with same principle as the big data clusters: we have more than one broker that replicates its content so if one broker fails the other assumes the lead and continues to transmit the data

Ok..enough with the theory, let's go to the project.

## Project

The goal in this project it's to transfer the data from a .csv file and put it on my hadoop cluster.

The first step it's to open streamsets. This tool works with Drag and Drop which means that very little programming knowledge is needed, we just have to select the blocks and then drop it on the interface.

After hit 'create a pipeline' I selected 'Directory' which means that I'll access a directory in my local machine(in this case the VM machine that Amazon provides). This directory will stores my json file. Then I'll create the path to this directory inside my Master Node. To do that I have to configure the Directory block on streamsets: first put the path of the json file, to help streamset locate it; second set the default file to json and last 'read order last modified datestamp' to order the data by its last read.
By the end of this configuration I had the following(the error in the image it's because I didn't connect the block with any other)

![workflow pipe](https://user-images.githubusercontent.com/68716835/104848064-03779380-58c2-11eb-8d61-58136b9e1fc4.PNG)

Now I have to connect with the Kafka Producer, to do that I had to install a library on streamsets that helps me deal with kafka. So I went to 'Packet Manager'(the gift image) wrote 'kafka' and installed the library. After the install I search for 'Kafka Producer' and linked it as Directory 1 destination. 

![kafka producer](https://user-images.githubusercontent.com/68716835/104851569-84d82180-58d4-11eb-86d4-e2b5a4da7050.PNG)


Now I have to initialize kafka on the Node Master, to do that I have first to initialize Zookeeper - to give a shallow explanation, Zookeeper is another Apache system and in this case helps to manage all the others Apache systems(since almost all the Apache system have an animal as its image you can think of zookeeper as the manager of the zoo, which it's true in real life).  

The following command initialize zookeeper:

```shell
nohup bin/zookeeper-server-start.sh config/zookeeper.properties > zookeeper.log &
```

* nohup: it's to run zookeeper in the background
* bin/zookeeper-server-start.sh config/zookeeper.properties : initialize zookeeper with specific properties
* '>' zookeeper.log: uses this file as the logfiles 
* &: to run in the background

Now I can initialize kafka:

```shell
nohup bin/kafka-server-start.sh config/server.properties > kafka.log &
```
The logic is the same as zookeeper so there's no need to explain it. The next step it's to create the kafka topic(that will receive the data):

```shell
bin/kafka-topics.sh --create --topic sensores --zookeeper localhost:2181 --replication-factor 1 --partitions 1
```
* bin/kafka.sh : starts kafka
* -- create --topic: creates topic and name it (sensores)
* --zookeeper: pass zookeepers location
* --replication-factor: the number of replications
* --partitions: number of partitions

After running this code, we can back to streamsets and configure the producer: changing the name of the topic to the one created before. Now, just hit validate on streamsets and then run

![validate successful](https://user-images.githubusercontent.com/68716835/104851587-b224cf80-58d4-11eb-99e0-b7a2ad887986.PNG)


Now back to the terminal..it's time to put the raw json file into the directory specified in the Directory 1. Once this is made I can back to streamsets and see if everything is all right. Every time a file is stored in the /data directory it'll be consumed for the Kafka Producer that will awaits to be sent to the consumer. Below the image showing the workflow consuming the data. 

![workflow funcionando](https://user-images.githubusercontent.com/68716835/104851774-c74e2e00-58d5-11eb-9279-6777d80ba039.PNG)

Since the streamset shows that we could be able to withdraw the data from its source. It's time to see with the consumer could be able to comunicate with the producer:

```shell
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic sensores --from-beginning
```
* bin/kafka-console-consumer.sh : starts kafka consumer
* --zookeeper: shows zookeeper location
* --topic: indicates the topic where the data are stored
* --from-beginning: reads them from the beginning

![arquivos consumidos](https://user-images.githubusercontent.com/68716835/104852208-4ba1b080-58d8-11eb-8e43-13748ee381e5.PNG)

Now that we saw that the data are consumed, we will create the next part of the pipeline. I'm going to build it in other streamset file to preserve reliability. If one block of the pipeline stops work it'll not affect the others. 

The Kafka Consumer will be configured in the same way as the Kafka Producer. But for the Hadoop block it'll be necessary to download the Hadoop library on packet manager. To configure hadoop I had to add where are the configuration files in my cluster,  in this case : /opt/hadoop/etc/hadoop, and on Hadoop FS URI use the default name on core-site.xml. 

But wait ... something is missing. I still have to clear the data! Therefore, between Kafka Consumer and HDFS Bloco, I will add the deduplication file. This file will help to delete duplicate lines from the file, as it has 2 outputs one for the hadoop block and the other for thrash. The top point is where clean data comes out and the other is where duplicate data comes out.

To configure the deduplicator just indicates which fields you want to be analyzed and then run the dataflow:

![removendoduplicadas](https://user-images.githubusercontent.com/68716835/104853326-d71e4000-58de-11eb-9be6-a453e4da642b.PNG)

![adiiconando deduplicador](https://user-images.githubusercontent.com/68716835/104853344-fddc7680-58de-11eb-8f2e-49202beab21b.PNG)

Once the streamset finishes the running process the data will appear on the HDFS directory specified earlier, and then the data will be ready to be consumed by the data scientists

![conteúdo no datalake](https://user-images.githubusercontent.com/68716835/104853366-22385300-58df-11eb-9a71-4c1973ea13fe.PNG)


