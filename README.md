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

## First..a little theory

Apache Kafka is a message system that has the goal to make data transfer between applications(spoiler: this is what we're going to do here). It works basically with three components:

* Producer: Responsible to produce the data, which means collect the data within the source and transfers it to the broker
* Broker: It's where the data is stored before gets consumed.
* Consumer: Reponsible to consume the data from the broker and connect it into another source

You might be asking yourself why do we need a tool that consumes data, store it, and then transfer it to other sources when I could only transfer the data between these two sources. And the answer is..reliability(among others of course). Let's say you're transfering information without Kafka and somehow you lost your internet connection. What will happened? Well, it's highly possible that your lost your work and have to transfer the data again. However with Kafka, since all the data comes first to the broker if your lost internet connection your work is saved into it and all you have to do it's reconnect and the data will begin to transfer.

The broker stores the data like a queue which means that every data is sent at a time(the first one to enters the queue is the first one to leave) but this happens at a very high speed so it looks almost instantly. Alright if my internet fails the broker can handle it, but what if the broker fails? Well that's why we have a **multi broker**. The multi broker works with same principle as the big data clusters: we have more than one broker that replicates its content so if one broker fails the other assumes the lead and continues to transmit the data
