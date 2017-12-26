# Practice Pig Latin

The aim of this repository is to give some hints for practicing/learning
Pig Latin. 

## Apache Pig

[Apache Pig](http://pig.apache.org/) is a platform for big data analysis.
It provides an SQL-like language called Pig Latin.

Because Hive and Spark are more comfortable for production, Pig is less
popular for big data analysis. But Pig is still useful because we can use
it without defining any schema for the data.

Imagine you have large CSV or JSON lines files on HDFS. To analyze the 
files with Apache Hive you have to define precisely the schema (column 
names and their data types). To use Spark you must have knowledge about
Scala, Java or Spark and Spark API.

Even though you have to learn Pig Latin, Apache Pig is comfortable in 
such a situation, because you do not have to define any schema. After 
loading the files, you can immediately analyze them.

## Contents of this repository

### 1. Reference of Pig Latin [pig-reference.md]
   
This is not a tutorial. If you are interested in getting 
[Hortonworks certification](https://hortonworks.com/services/training/certification/hdpcd-certification/),
then it is enough to understand the concepts in the reference.

### 2. Graph Analysis on local [pig-local.md]
   
The last assignment of the online course [Communicating Data Science Results](https://www.coursera.org/learn/data-results)
is a big data analysis with Pig Latin on AWS. This tutorial explains how to 
practice Pig Latin for the assignment on a local machine, so that you do 
not have to worry about charging. 






