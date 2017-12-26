# Graph Analysis on local 

This is a tutorial for practicing Pig Latin for the programming assignment 
"[Graph Analysis in the Cloud](https://github.com/uwescience/datasci_course_materials/blob/master/assignment4/assignment4.md)"
of the online course 
"[Communicating Data Science Results](https://www.coursera.org/learn/data-results/)",
which is the third course of the specialization 
"[Data Science at Scale](https://www.coursera.org/specializations/data-science)". 

- **The original instruction of the instructor is out-of-date.**
  You have to check a thread 
  "[Revised Instructions for AWS Assignment](https://www.coursera.org/learn/data-results/discussions/weeks/3/threads/BKIe94xzEea29BIY1fPlKQ)"
  in a discussion forum.
- Fundamentals of Pig Latin and graph analysis are explained in Week 4 of 
  the first course of the coursera specialization:
  "[Data Manipulation at Scale: Systems and Algorithms](https://www.coursera.org/learn/data-manipulation/)".
- There is no answer to questions of the assignment. 

The reader of this tutorial is supposed to 

- understand Linux commands and 
- have accomplished the first course of the specialization.

## Background 

In the assignment we analyze big data on AWS.

Even if you get AWS Promotional Credit after completing the course, it is
quite uncomfortable to work on AWS so long you have to pay the charge.
But you do not have to use AWS to learn Pig (and to try scripts provided
by the instructor.)

In this tutorial we explain briefly how to practice Pig Latin on a local
machine, i.e. without AWS, in order to complete the assignment.

Buy the way I paid around $40 when I did the assignment. In several months 
after the completion of the course I obtained the promotional credit with 
$50. I could not be payed back, but I made use of it when I practiced for 
a Hortonworks certification.

## Setup 

To use Apache Pig on a local machine there are roughly two ways.

1. Install Pig on a local machine. 
2. Use a certain virtualbox image or docker component on which Pig 
   (and Hadoop system) is available. 

### 1st option: Install Pig 

If you work on a Unix/Linux machine, the first option is easiest. 
Download the complied package and unpack it. Then you can use it 
immediately (if Java is already installed). You do not have to 
install Apache Hadoop on your machine, you do not have to set 
environmental variable, neither.

The working directory looks like this:

	├── pig/
	│   ├── bin/
	│   ├── build.xml
	│   ├── CHANGES.txt
	│   ├── conf/
	│   ├── contrib/
	│   ├── docs/
	│   ├── ivy/
	│   ├── ivy.xml
	│   ├── legacy/
	│   ├── lib/
	│   ├── lib-src/
	│   ├── license/
	│   ├── LICENSE.txt
	│   ├── NOTICE.txt
	│   ├── pig-0.16.0-core-h1.jar
	│   ├── pig-0.16.0-core-h2.jar
	│   ├── README.txt
	│   ├── RELEASE_NOTES.txt
	│   ├── scripts/
	│   ├── shims/
	│   ├── src/
	│   ├── test/
	│   └── tutorial/
	├── pig-0.16.0.tar.gz
	└── workspace/

Unpacking `pig-0.16.0.tar.gz` you get the directory `pig` and files under
it.

Move to the `workspace` directory and execute `../pig/bin/pig --version` 
on the shell. Then you might get the following message. 

	which: no hadoop in (...)
	Apache Pig version 0.16.0 (r1746530) 
	compiled Jun 01 2016, 23:09:59

`...` is just an omitted part. 

Let's try a sample script. Create a directory `tutorial1` under `workspace`
and copy files we need:

	$ mkdir tutorial1 
	$ cp ../pig/tutorial/scripts/script1-local.pig tutorial1/
	$ cp ../pig/tutorial/data/excite-small.log tutorial1/
	$ cp ../pig/tutorial/build/output/pigtmp/tutorial.jar tutorial1/

Then we can execute the copied script 

	$ cd tutorial1 
	$ ../../pig/bin/pig -x local script1-local.pig 

After a few minutes you obtain a *directory* `script1-local-results.txt`.

	├── excite-small.log
	├── pig_1514303185032.log
	├── script1-local.pig
	├── script1-local-results.txt/
	│   ├── part-r-00000*
	│   └── _SUCCESS*
	└── tutorial.jar

`part-r-00000` is a TSV (Tab-Separated Values) file which contains 
the results of the analysis.

### 2nd option: Use VirtualBox or Docker

If you do not work on a Unix/Linux machine, you might want to try 
Virtualbox image or Docker component on which Hadoop system is installed.

For example Hortonworks provides a sandbox 
[HDP](https://hortonworks.com/downloads/) and tutorials including
"[Beginners Guide to Apache Pig](https://hortonworks.com/tutorial/beginners-guide-to-apache-pig/)".

For installation you should follow the instruction provided by Hortonworks
and you might want to read the tutorials after the installation to 
understand how the sandbox works.

## Try the sample script. 

Apache Pig can work without Hadoop, if it is executed in the local mode.
So just use it.

Assume that we have taken the first option above. 

Let's try the sample script `example.pig` provided by the course instructor.
You have already copied/cloned the repository of the course, haven't you? 
Then you create a directory `assignment` under `workspace`. Copy 
`example.pig` and `myudfs.jar` in `assignment` directory. Download 
[cse344-test-file](http://uw-cse-344-oregon.aws.amazon.com.s3.amazonaws.com/cse344-test-file)
and save it in the same directory. Now `assignment` directory contains 
three files.

	├── cse344-test-file
	├── example.pig
	└── myudfs.jar

Next we modify the script `example.pig` so that it works on local:

- Change the path to the JAR file in the `register` statement:

    	register ./myudfs.jar

- Change the path to the data in the `load` statement:

	    raw = LOAD 'cse344-test-file' USING TextLoader as (line:chararray);

- Remove "PARALLEL 50" from the `order` statement: 

	    count_by_object_ordered = order count_by_object by (count);

- Change the path in the `store` statement: 

    	store count_by_object_ordered into 'example-results' using PigStorage();

After changing the script we can execute the script:

	$ ../../pig/bin/pig -x local example.pig
	
(If you are working on a HDP, you should obviously replace 
`../../pig/bin/pig` with `pig`.)

After a few minutes you obtain the results under the directory 
`example-results`. 

Notes: 

- The directory `example-results` should not exist when executing the
  script. 
- When executing the script on AWS you have to throw away the above 
  modification. 
- Have a fan!
