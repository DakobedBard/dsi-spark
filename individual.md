###1 Spark Basics

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
    
4. To retrieve items in your RDD into a Python list, you would have to access every
   partition of the RDD and this could take a long time. Before you execute the following 
   command, be aware of how many entries you are pulling. Usually you pool the results 
   (of reasonable size) into a Python list after all the big data operations are done in RDDs.
   
   ```python
   file_rdd.collect()
   lst_rdd.collect()
   ```

###2. Functional Programming

Spark operations conforms to the functional programming paradigm. Objects (RDDs) are immutable 
and mapping a function to an RDD returns another RDD. A lot of Spark's functionalities assume the 
items in an RDD to be tuples of `(key, value)`. Structure your RDDs to be `(key, value)` whenever possible.

1. Turn the items in `file_rdd` into `(key, value)` pairs. Map each item into a json object and then map to
   the `(key, value)` pairs. **Remember to cast value as type** `int`.
   
   - **The key is the name of the person**
   - **The value is how many chocolate chip cookies bought**

    
2. Filter Similiar to `map`, `filter` the entries in the 
   `json_rdd` with more than 5 chocolate chip cookies.

3. Most Spark built-in functions assumes each item in the RDD is a tuple of 2 `(key, value)`.
   Use `map` again on `json_rdd` to make each item a tuple, i.e. `(name, cookie bought)`. 
   Run a `.first()` to confirm your results.
   
   **Note:**
   
   **All your map functions are not run in the Spark backend when you execute it.
   The map operations are run when a `.first()`, `.take()` or `.count()` is called where the items
   are needed. This is known as [lazy evaluation](http://en.wikipedia.org/wiki/Lazy_evaluation)**

4. Now we are interested in calculating how much people purchased for their cookies. Use `mapByValue`
   to return an RDD with tuples `(name, money purchased)`. Again run a `first()` to confirm.

5. Make the names lower-case, use `mapByKey` to achieve that.

   **Note:**
   
   **In general, when you are doing functional programming with Spark, keep the function of 
   each of your map operation small. Spark will be able to distribute resources and parallelize 
   more efficiently**
   
6. Same as mrjobs of the map-reduce framework, Spark is built on the same framework and can do
   reduce operations. Calculate the total amount of money from purchasing cookies from everyone.
   Use `rdd.values()` to access all the values as an RDD and then do a `reduce` to get the sum.
   This might seem to be a rather pointless exercise with a few entries in this toy dataset, but
   the performance difference is huge on massive data sets.
   
   
##2. Practical Operations with Spark

Now we are familiar with the basics of Spark. Let's flex some of Spark's real power with a bigger
dataset. Here we will run through the workflow of working with an actual dataset in Spark.
This dataset is still not so huge that it will be impossible to process locally. We will
get to the real big ones tomorrow where we need to be running operations on cluster of machines in
the cloud.

Here we are trying to find out which airport has the worst / least delay. There are 2 types of delays:
arrival delays (`ARR_DELAY`) and departure delays (`DEP_DELAY`). All delays are in terms of minutes.
`ARR_DELAY` is associated with the destination airport (`DEST_AIRPORT_ID`), and
`DEP_DELAY` is associated with the destination airport (`ORIGIN_AIRPORT_ID`).

1. Just as above, define your `SparkContext` to be `local[4]`. Load the text file in via an S3 link 
   to the file. 

   ```python
   airline = sc.textFile('s3n://mortar-example-data/airline-data')
   ```

2. Show the first 2 entries. The first line is the column names and starting from the second line is 
   the corresponding data. Also run a `.count()` on the RDD. This will **take a while** as the data set is
   a few million rows.

3. Let's do some preprocessing. Remove the `'`, `"` and the trailing `,` for each line. Show the first 2 lines
   to confirm. The first 2 lines should look like the following.
   
   ```
   YEAR,MONTH,UNIQUE_CARRIER,ORIGIN_AIRPORT_ID,DEST_AIRPORT_ID,DEP_DELAY,DEP_DELAY_NEW,ARR_DELAY,ARR_DELAY_NEW,CANCELLED
   2012,4,AA,12478,12892,-4.00,0.00,-21.00,0.00,0.00
   ```

4. Run a count on the RDD. Notice the size of the data you are dealing with.
  
5. Use `filter` to filter out the line containing the column names. 


6. Make a function, `make_rows()`, that takes a line as an argument and return a dictionary
   where the keys are the column names and the values are the values for the column. 
   
   - The output is a dictionary with only these columns:
     `['DEST_AIRPORT_ID', 'ORIGIN_AIRPORT_ID', 'DEP_DELAY', 'ARR_DELAY']`
   - Cast `DEP_DELAY` and `ARR_DELAY` as an integer. These are minutes that are delayed.
   - Subtract `DEP_DELAY` from `ARR_DELAY` to get the actual `DEP_DELYAY`
   - If a flight is `CANCELLED`, add 5 hours to `DEP_DELAY`
   - There are missing values in `DEP_DELAY` and `ARR_DELAY` (i.e. `''`) and you would want
     to replace those with `0`.
     
   Map `make_rows()` to the RDD and you should have an RDD where each item is a dictionary.
   
7. Instead of dictionaries, make 2 RDDs where the items are tuples.
   The first RDD will contain tuples `(DEST_AIRPORT_ID, ARR_DELAY)`. 
   The other RDD will contain `(ORIGIN_AIRPORT_ID, DEP_DELAY)`.
   Run a `.first()` or `.take()` to confirm your results.

8. Make 2 RDDs for the mean delay time for origin airports and destination airports. You will need 
   to `groupByKey()` and then take the mean of the delay times for the particular airport. 
   Use the PySpark [docs](http://spark.apache.org/docs/latest/api/python/pyspark.html#module-pyspark).

9. Run `rdd.persist()` on the RDDs you made in in `8.`. That will cache the RDDs so they do not
   need to be reproduced every time they are called upon. Use `persist()` for RDDs that you are 
   going to repeatedly use.

10. Sort the RDDs by the mean delay time to answer the following questions.

    - Top 10 departing airport that has least avgerage delay in minutes
    - Top 10 departing airport that has most avgerage delay in minutes
    - Top 10 arriving airport that has least avgerage delay in minutes
    - Top 10 arriving airport that has most avgerage delay in minutes

   

