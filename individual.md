##1. RDD and Spark Basics

Here we will be using **PySpark**, but the concepts are transferrable to Spark in Scala.

1. Initiate a `SparkContext`. A `SparkContext` specifies where your
   cluster is, i.e. the resources for all your distributed computation. Specify your `SparkContext`
   as follows.
   
   ```python
   import pyspark as ps
   # Uses all 4 cores on your machine
   sc = ps.SparkContext('local[4]') 
   ```

2. Spark operates in **Resilient Distributed Datasets (RDDs)**.
   **An RDD is a collection of data partitioned across machines**. 
   Using RDDs allow the processing of your data to be parallelized due to the partitions.
   RDDs can be created by referencing datasets in external storage systems, or from Python
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
   the memory. You can quickly check out the first few entries of the RDD
   of a potentially enormous data set without accessing all the partitions.
    
   ```python
   file_rdd.first() # Views the first entry
   file_rdd.take(2) # Views the first two entries
   ```
    
4. To retrieve all items in your RDD into a Python list, every partition in the RDD has to
   accessed and this could take a long time. In general, before you execute the following 
   command, be aware of how many entries you are pulling. Usually you pool the results 
   (of reasonable size) into a Python list after all the big data operations are done in RDDs.
   
   ```python
   file_rdd.collect()
   lst_rdd.collect()
   ```

##2. Intro to Functional Programming

Spark operations conforms to the functional programming paradigm. Objects (RDDs) are immutable 
and mapping a function to an RDD returns another RDD. A lot of Spark's functionalities assume the 
items in an RDD to be tuples of `(key, value)`. Structure your RDDs to be `(key, value)` whenever possible.

Also beware of [**lazy evaluation**](http://en.wikipedia.org/wiki/Lazy_evaluation) where operations
are not executed until a `.collect()`, `.first()`, `.take()` or `.count()` is call to retrieve items
in the RDD.

1. Turn the items in `file_rdd` into `(key, value)` pairs using `map()` and a `lambda` function. Map each item into a json object and then map to
   the `(key, value)` pairs. **Remember to cast value as type** `int`. Use `collect()` to see your results.
   Using `collect()` is fine here since the data is small.
   
   - **The key is the name of the person**
   - **The value is how many chocolate chip cookies bought**
    
2. Now use `filter()` to look for entries with more than `5` chocolate chip cookies.

3. For each name, return the entry with the max number of cookies. 
   
   **Hint:** 
   - Use `groupByKey()`, `mapValues()`
   - Use `iterable.data` to convert a `pyspark.resultiterable.ResultIterable` to a Python list
 
4. Calculate the total revenue from people buying cookies.

   **Hint:**
   - `rdd.values()` returns another RDD of all the values
   - Use `reduce()` to return the sum of all the values
      
   
##3. Processing Data with Spark

Now let's scale up to a larger dataset.

### Objectives

* Identify airports with the worst / least delay.
 
**2 types of delays:**

- Arrival delays (`ARR_DELAY`) and departure delays (`DEP_DELAY`)
- All delays are in terms of **minutes**
- `ARR_DELAY` is associated with the destination airport (`DEST_AIRPORT_ID`)
- `DEP_DELAY` is associated with the destination airport (`ORIGIN_AIRPORT_ID`)

1. Start a new notebook and make a new `SparkContext`. There could only be one Spark instance
   per Python instance. Load the file as follow.

   ```python
   airline = sc.textFile('s3n://mortar-example-data/airline-data')
   ```

2. Print the first 2 entries. The first line is the column names and starting from the second line is 
   the corresponding data. Also run a `.count()` on the RDD. This will **take a while** as the data set is
   a few million rows.

3. Let's do some preprocessing. Remove the `'`, `"` and the trailing `,` for each line. Print the first 2 lines
   to confirm. The first 2 lines should look like the following.
   
   ```
   YEAR,MONTH,UNIQUE_CARRIER,ORIGIN_AIRPORT_ID,DEST_AIRPORT_ID,DEP_DELAY,DEP_DELAY_NEW,ARR_DELAY,ARR_DELAY_NEW,CANCELLED
   2012,4,AA,12478,12892,-4.00,0.00,-21.00,0.00,0.00
   ```
  
4. Use `filter` to filter out the line containing the column names. 


5. Make a function, `make_rows()`, that takes a line as an argument and return a dictionary
   where the keys are the column names and the values are the values for the column. 
   
   - The output is a dictionary with only these columns:
     `['DEST_AIRPORT_ID', 'ORIGIN_AIRPORT_ID', 'DEP_DELAY', 'ARR_DELAY']`
   - Cast `DEP_DELAY` and `ARR_DELAY` as an integer. These are minutes that are delayed.
   - Subtract `DEP_DELAY` from `ARR_DELAY` to get the actual `ARR_DELYAY`
   - If a flight is `CANCELLED`, add 5 hours to `DEP_DELAY`
   - There are missing values in `DEP_DELAY` and `ARR_DELAY` (i.e. `''`) and you would want
     to replace those with `0`.
     
   Map `make_rows()` to the RDD and you should have an RDD where each item is a dictionary.
   
6. Instead of dictionaries, make 2 RDDs where the items are tuples.
   The first RDD will contain tuples `(DEST_AIRPORT_ID, ARR_DELAY)`. 
   The other RDD will contain `(ORIGIN_AIRPORT_ID, DEP_DELAY)`.
   Run a `.first()` or `.take()` to confirm your results.

7. Make 2 RDDs for the mean delay time for origin airports and destination airports. You will need 
   to `groupByKey()` and then take the mean of the delay times for the particular airport. 
   Use the PySpark [docs](http://spark.apache.org/docs/latest/api/python/pyspark.html#module-pyspark).

8. Run `rdd.persist()` on the RDDs you made in in `8.`. That will cache the RDDs so they do not
   need to be reproduced every time they are called upon. Use `persist()` for RDDs that you are 
   going to repeatedly use.

9. Sort the RDDs by the mean delay time to answer the following questions.

    - Top 10 departing airport that has least avgerage delay in minutes
    - Top 10 departing airport that has most avgerage delay in minutes
    - Top 10 arriving airport that has least avgerage delay in minutes
    - Top 10 arriving airport that has most avgerage delay in minutes

   

