##Spark Basics

PySpark is a Python wrapper around Spark, which is written in Scala. The following concepts are 
applicable to Spark and PySpark. Complete the following tutorial in  

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

3. first collect take 





##Power of Spark

The value of Spark is its ability to process data at a capacity that Python cannot. Here we are going 