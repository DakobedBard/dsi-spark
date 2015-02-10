##1. Intro to Spark
PySpark is a Python wrapper around Spark, which is written in Scala. The following concepts are 
applicable to Spark and PySpark. Complete the following tutorials in **Spark Basics** 
and **Functional Programming**. 


###1.1 Spark Basics

1. Before you start using PySpark, you need a `SparkContext`. A `SparkContext` specifies where your
   cluster is, i.e. the resources for all your distributed computation. Specify your `SparkContext`
   as follows.
   
   ```python
   import pyspark as ps
   sc = ps.SparkContext('local[4]') # Used all 4 cores on your machine
   ```

2. The fundamental programming abstraction in Spark is called Resilient Distributed Datasets (RDDs).
   An RDD is a logical collection of data partitioned across machines. RDDs can be created by 
   referencing datasets in external storage systems, or by applying coarse-grained transformations 
   (e.g. map, filter, reduce, join) on existing RDDs.
   
   Create an RDD from a python list. 
   
   ```python
   lst_rdd = sc.parallelize([1, 2, 3])
   ```
   
   Read an RDD in from a text file. By default, the RDD will treat each line as an item.
   
   ```python
   file_rdd = sc.textFile('data/toy_data.txt')
   ```

3. Now we have an RDD, we need to see what is inside. RDD by default will load data in
   partition. Therefore at creation of the RDD, the data is not completely loaded onto
   the memory. This way you are able to quickly check out the first few entries of the RDD
   of a potentially enormous data set.
    
   ```python
   file_rdd.first() # Views the first entry
   file_rdd.take(2) # Views the first two entries
   ```
    
   If you would like to have everything in a Python list, you would have to access every
   partition of the RDD and this could take a long time. Before you execute the following 
   command, be aware of how many entries you are pulling. It is completely fine for this 
   toy data set.
   
   ```python
   file_rdd.collect()
   lst_rdd.collect()
   ```

###1.2 Functional Programming

4. 

##2. Practical Operations with Spark


##Power of Spark

The value of Spark is its ability to process data at a capacity that Python cannot. Here we are going 