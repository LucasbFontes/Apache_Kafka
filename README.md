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

Ok..enough with the theory, let's go to the project.

## Project

The goal in this project it's to transfer the data from a .csv file and put it on my hadoop cluster. I'm going to show 2 projects: first I'm going to show how I move the data from one point to other; and in the second project I'll do the same  but cleaning the data before stores it on Hadoop.

The first step it's to open streamsets. This tool works with Drag and Drop which means that very little programming knowledge is needed, we just have to select the blocks and then drop it on the interface.

After hit 'create a pipeline' I selected 'Directory' which means that I'll access a directory in my local machine(in this case the VM machine that Amazon provides). This directory will stores my json file. Then I'll create the path to this directory inside my Master Node. To do that I have to configure the Directory block on streamsets: first put the path of the json file, to help streamset locate it; second set the default file to json and last 'read order last modified datestamp' to order the data by its last read.
By the end of this configuration I had the following(the error in the image it's because I didn't connect the block with any other)

![workflow pipe](https://user-images.githubusercontent.com/68716835/104848064-03779380-58c2-11eb-8d61-58136b9e1fc4.PNG)



