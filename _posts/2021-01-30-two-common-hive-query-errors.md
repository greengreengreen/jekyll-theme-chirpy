---
title: Two Common Hive Query Errors
date: 2021-01-30 16:00:00 -0800
categories: [Big Data, Hive]
tags: [debug, hive queries]     # TAG names should always be lowercase
---

As a data scientist, I use Hive SQL almost everyday. When I first used Hive, I spent lots of time debugging and understanding my Hive query errors. If you are in the same situation, this article is for you:) 

The major difference between Hive and relational databases like MySQL is that Hive is based on distributed system. Most of time, Hive is running within the Hadoop ecosystem, which means it uses HDFS for file storage and MapReduce for computing. 

Below, I will try to illustrate two common Hive query errors based on Hive's underlying mechanism using HDFS and MapReduce. 


## 🤒 Symptom1: Query failed before any map/reduce task starts. Log says “exceeds # of task limit"
### 🤔 Why? 

In a Hadoop cluster, there is a limited number of processes can be running spontaneously. This can be configured by cluster admin by setting “org.apache.hadoop.mapred.JobInProgress.initTasks”.  Since Hive query is computed by being partitioned into multiple map/reduce tasks, each task is a process. If you set the the split size to be too small, your query job will be splitted into many tasks. Having more tasks will definitely increase the parallelism of the query and accelerate the running time. However, if the number of tasks is too big and exceed the number of task limit, then it will be killed by the cluster. 
### 💊 Solution: 

Set mapred.max.split.size to be smaller. 

## 🤒 Symptom2: The query exits after a few maps/reduces tasks running. Log says “disk quota exceeded"
### 🤔 Why? 

First, congrats! If your query has successfully kicked out the map/reduce processes, then you have no syntax error or task splitting error. What is happening is that your intermediate result has exceeded the disk space. Map/Reduce task runs in a datanode. In order for the datanode to deliver the result data into HDFS, it will first need to write the results in its local disk. Although the HDFS normally has adequate space, a datanode may not have enough local disk space or it is configured to have limited space for each user/task. A complex query could generate lots of data in between, which exceeds the disk space of a datanode and kills the query. 
### 💊 Solution: 
<ol>
<li> Clean the data node for more space. </li>
<li> Run the same query for smaller chunk of data. </li>
<li> Replace the complex query with several simple queries. </li>
</ol>

A Hive query can fail for many reasons. Sometimes, a random node failing could have stopped the query. In addition to the above symptoms which have clear solutions, I would also test out a very simple query to see whether or not it is a query-dependent error. 

Hope this is helpful!

