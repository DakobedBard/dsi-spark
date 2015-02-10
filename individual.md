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
   sc = ps.SparkContext('local[4]') # Uses all 4 cores on your machine
   ```

2. The fundamental programming abstraction in Spark is called **Resilient Distributed Datasets (RDDs)**.
   **An RDD is a logical collection of data partitioned across machines**. This is where most of
   Spark's power come from. Therefore, you would want to use RDDs whenever you are processing massive 
   amounts of data. RDDs can be created by referencing datasets in external storage systems, or from Python
   list objects.
   
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

1. All operations in Spark are functional. In other words, you map functions to each item in 
   your RDD. You will rarely need a for loop (`forEach()`) in Spark. 
   
   As it stands now, each line in `file_rdd` is a string object, below is an example of 
   how to map a function to each line to turn the line into a dictionary.
   
   ```python
   json_rdd = file_rdd.map(lambda line: json.loads(line))
   json_rdd.first()
   ```

   Now we have dictionaries within RDD which we can work with.

2. The key of the dictionary is the name of the person and the key is how many chocolate chip 
   cookies they have bought for the past month. Similiar to `map`, `filter` the entries in the 
   `json_rdd` with more than 5 chocolate chip cookies.

3. Most Spark built-in functions assumes each item in the RDD is a tuple of 2 `(key, value)`.
   Use `map` again on `json_rdd` to make each item a tuple, i.e. `(name, cookie bought)`. 
   Run a `.first()` to confirm your results.
   
   **Note that all your map functions are not run in the Spark backend when you execute it.
   The map operations are run when a .first(), take() or .count() is called where the items
   are needed. This is known as [lazy evaluation](http://en.wikipedia.org/wiki/Lazy_evaluation)**

3. Now we are interested in calculating how much people purchased for their cookies. Use `mapByValue`
   to return an RDD with tuples `(name, money purchased)`. Again run a `first()` to confirm.



##2. Practical Operations with Spark


##Power of Spark

The value of Spark is its ability to process data at a capacity that Python cannot. Here we are going 