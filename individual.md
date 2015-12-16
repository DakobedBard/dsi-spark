##Part 1: RDD and Spark Basics

Here we will get familiar with the basics of Spark via the Spark Python API 
(`PySpark`). For now, we will be just working with a master/driver node that 
will parallelize its work across all our cores (rather than handing off the
work to its workers). In `Part 3`, we are going to simulate a master-worker 
cluster to run our jobs.

1. Initiate a `SparkContext`. A `SparkContext` specifies where your cluster is,
   i.e. the resources for all your distributed computation. Specify your 
   `SparkContext` as follows.
   
   ```python
   import pyspark as ps
   # Uses all 4 cores on your machine
   sc = ps.SparkContext('local[4]') 
   ```

   **Note**: You may not have 4 cores on your machine, and if you don't you should 
   adjust how many cores you are telling the SparkContext to use. To find out 
   how many you have, you can run: 
   
   ```python
   import multiprocessing
   multiprocessing.cpu_count()
   ```

2. Spark operates in **[Resilient Distributed Datasets (RDDs)][RDDs]. An RDD is 
   a collection of data partitioned across machines**. Using RDDs allows the 
   processing of your data to be parallelized due to the partitions. RDDs can 
   be created from your SparkContext in two ways: loading an external dataset, 
   or by parallelizing an existing collection of objects in your currently 
   running program (in our Python programs, this is often times a list). 
   
   Create an RDD from a python list. 
   
   ```python
   lst_rdd = sc.parallelize([1, 2, 3])
   ```
   
   Read an RDD in from a text file. **By default, the RDD will treat each line 
   as an item and read it in as string.**
   
   ```python
   file_rdd = sc.textFile('data/cookie_data.txt')
   ```

3. Now that we have an RDD, we need to see what is inside. RDDs by default will 
   load data into partitions, rather than loading it into memory. This means that
   you can quickly check out the first few entries of a potentially enormous RDD
   without accessing all of the partitions and loading all of the data into memory.
    
   ```python
   file_rdd.first() # Views the first entry
   file_rdd.take(2) # Views the first two entries
   ```
    
4. To retrieve all the items in your RDD, every partition in the RDD has to be 
   accessed, and this could take a long time. In general, before you execute 
   commands (like the following) to retrieve all the items in your RDD, you 
   should be aware of how many entries you are pulling. Keep in mind that to 
   execute the `collect()` method on the RDD object (like we do below), your entire 
   dataset must fit in memory on a single machine (we in general don't want 
   to call `collect()` on very large datasets). The standard workflow when working 
   with RDDs is to perform all the big data operations/transformations 
   **before** you pool/retrieve the results. If the results can't be `collect()ed`
   onto a single machine, it's common to write data out to a distributed storage
   system, like HDFS or S3. 

   With all that said, we can retrieve all the items from our RDD as follows: 
   
   ```python
   file_rdd.collect()
   lst_rdd.collect()
   ```

##Part 2: Intro to Functional Programming

Spark operations conform to a [functional programming paradigm][funct-programming]. 
In terms of our RDD objects, what this means is that our RDD objects are immutable 
and that anytime we apply a **transformation** to an RDD (such as `map()`, `reduce()`,
or `filter()`) this transformation returns another RDD. 

**Spark notes**:

   * A lot of Spark's functionalities assume the items in an RDD to be tuples 
   of `(key, value)` pairs, so often times it can be useful to structure your 
   RDDs this way. 
   * Beware of [**lazy evaluation**][wiki-lazy-eval], where tranformations 
   on the RDD are not executed until an **action** is executed on the RDD 
   to retrieve items from it (such as `collect()`, `first()`, `take()`, or 
   `count()`). So if you are doing a lot transformations in a row, it can 
   be helpful to call `first()` in between to ensure your transformations are 
   running properly.
   * If you are not sure what RDD transformations/actions there are, you can 
   check out the [docs][RDD-docs].

1. Turn the items in `file_rdd` into `(key, value)` pairs using `map()` and a 
`lambda` function. Map each item into a json object (use `json.loads()`) and 
then map the json object to a `(key, value)` pair. **Remember to cast 
value as type** `int`.  Use `collect()` to see your results. Using `collect()` 
is fine here since the data is small. Make sure that: 
   
   - **The key is the name of the person**
   - **The value is how many chocolate chip cookies they bought**
    
2. Now use `filter()` to look for entries with more than `5` chocolate chip cookies.

3. For each name, return the entry with the max number of cookies. 
   
   **Hint:** 
   - Use `reduceByKey()` instead of `groupByKey()`. See why [here][groupby-v-reduceby-key]. 

4. Calculate the total revenue from people buying cookies (we're assuming that
each cookie only costs $1). 

   **Hint:**
   - `rdd.values()` returns another RDD of all the values
   - Use `reduce()` to return the sum of all the values

##Part 3: Starting a Local Cluster

Here we will simulate starting a master/worker cluster locally. This is different
from parts 1 and 2 in that there will now be workers with which the master will 
communicate. Simulating a master/worker cluster is useful in that it allows us 
to develop code on a local cluster before deployment. This way, we can work out 
any of the kinks/problems that might be in our programs before using valuable 
cluster resources. 

We will be using [`tmux`](http://tmux.github.io/) to run our scripts in 
the background. `tmux` lets us *multiplex* our terminal, create terminal sessions,
and attach/detach different programs in the terminal (somewhat like running 
processes in hidden terminals). This will be useful for us because we will be 
able to push our cluster into the background and focus on our python programs 
that are going to do work on the cluster.

Here is a **quick** guide to tmux for you to skim through: 

```bash 
brew install tmux # Install tmux. 
tmux new -s [session_name] # Start a new tmux session. 
ctrl + b, d # Detach from that tmux session. 
tmux ls # Get a list of your currently running tmux sessions. 
tmux attach -t [session_name] # Attach to an existing session. 
``` 

<br>  

Now let's use `tmux` to create a local cluster (master and workers) which will 
be running in terminals in the background. 

1. Start a tmux session which will host your master node:

   ```bash
   tmux new -s master
   ```
2. Run the following command to set up the Spark master to listen on local IP. 
The Master class in `org.apache.spark.deploy.master` accepts the following 
parameters: 
   
   - `h` : host (which on our case is local host `127.0.0.1`) 
   - `p`: The port on which the master is listening in (`7077`)
   - `webui-port`: The port on which the webui is reachable (`8080`)
   
   <br>

   ```bash
   ${SPARK_HOME}/bin/spark-class org.apache.spark.deploy.master.Master \
   -h 127.0.0.1 \
   -p 7077 \
   --webui-port 8080
   ```
3. You should get some output in your terminal similar to the following:
   ![master_term](https://github.com/zipfian/spark/blob/master/images/master_term.png)

4. Detach from your master session(`crtl+b, d`). Start a new tmux session which 
   will host your first worker node: 
   
   ```bash
   tmux new -s worker1
   ```

5. Start a worker by running the following:

   ```bash
   ${SPARK_HOME}/bin/spark-class org.apache.spark.deploy.worker.Worker \
   -c 1 \
   -m 1G \
   spark://127.0.0.1:7077
   ```
   
   This will start a worker with 1GB memory and 1 core and attach it to the 
   previously created Spark master. The output in your terminal should be:
   
   ![worker_term](https://github.com/zipfian/spark/blob/master/images/worker_term.png)

   Detach the current session and create a new session and run the same command 
   to create a second worker. 

6. You have now set up a master with 2 workers locally. Spark also provides us 
   with a web UI that lets us track the Spark jobs and see other stats about 
   any Spark related tasks and workers. 

   <h3 style="color:red">Your web UI is at: <code>localhost:8080</code></h3>

   ![sparkui_first](https://github.com/zipfian/spark/blob/master/images/sparkui_first.png)

7. We are not running any applications with our local Spark cluster yet. 
   We can attach an IPython notebook to the master and start `pyspark` by 
   running the following command. This will start the notebook in the browser
   and assign 1G of RAM per executor (worker node) and 1G of RAM to the master 
   in our pyspark application. Then, we can interact with `pyspark` via a 
   SparkContext, just like we did in parts 1 and 2 (the difference here is that
   we actually have worker nodes that our master will communicate with and 
   assign tasks to). 

   ```bash
   IPYTHON_OPTS="notebook"  ${SPARK_HOME}/bin/pyspark \
   --master spark://127.0.0.1:7077 \
   --executor-memory 1G \
   --driver-memory 1G
   ```

8. A SparkContext has already been loaded into IPython. Access it via the 
variable `sc`.

   You will see an output like below:

   ```python
   sc
   pyspark.context.SparkContext at 0x104318250
   ```

9. Now if you refresh your spark web UI, you should see **`PySparkShell`** running in the list of applications. 
   
  ![running_application](https://github.com/zipfian/spark/blob/master/images/running_application.png)

##Part 4: Spark for Data Processing 

Using the cluster we have set up in `Part 3`, we will be dealing with airport data and 
we want to identify airports with the worst / least delay.
 
**2 types of delays:**

- Arrival delays (`ARR_DELAY`) and departure delays (`DEP_DELAY`)
- All delays are in terms of **minutes**
- `ARR_DELAY` is associated with the destination airport (`DEST_AIRPORT_ID`)
- `DEP_DELAY` is associated with the destination airport (`ORIGIN_AIRPORT_ID`)

<br>

1. Read [**lectures/sparkui.md**](lectures/sparkui.md) for further guide as to how to use 
   the UI. The guide will bring you through `2.` and `3.`. 
   
   *Note*: If you have issues with proxy settings and you're using Firefox, try 
   using Chrome. 

2. Load the file in as follows. (Note: Loading won't work if your AWS keys contain 
a slash. Generate a new pair if necessary.)

   ```python
   # DON'T INCLUDE THE '[' AND ']' when you replace your keys. 
   link = 's3n://[YOUR_AWS_ACCESS_KEY_ID]:[YOUR_AWS_SECRET_ACCESS_KEY]@mortar-example-data/airline-data'
   airline = sc.textFile(link)
   ```
   
3. Print the first 2 entries. The first entry is the column names and starting 
   with the second we have our data. Also run a `.count()` on the RDD. This will 
   **take a while**, as the data set is a few million rows. 

4. As you can see, `.count()` takes a long time to run. More involved commands can 
   take even longer. In order not to waste time when writing/testing your code, 
   it's common practice to work with a sub-sample of your data until you have 
   your code finalized/polished and ready to run on the full dataset. Use 
   `.take(100)` to sample out the first 100 rows and assign it to a new RDD 
    using `sc.parallelize`.

5. Let's do some preprocessing. Remove the `'`, `"` and the trailing `,` for 
   each line. Print the first 2 lines to confirm. The first 2 lines should 
   look like the following.
   
   ```
   YEAR,MONTH,UNIQUE_CARRIER,ORIGIN_AIRPORT_ID,DEST_AIRPORT_ID,DEP_DELAY,DEP_DELAY_NEW,ARR_DELAY,ARR_DELAY_NEW,CANCELLED 
   2012,4,AA,12478,12892,-4.00,0.00,-21.00,0.00,0.00
   ```
  
6. Use `filter` to filter out the line containing the column names. 

7. Write a function, `make_rows()`, that takes a line as an argument and returns
   a dictionary where the keys are the column names and the values are the 
   values for the column. 
   
   - The output is a dictionary with only these columns:
     `['DEST_AIRPORT_ID', 'ORIGIN_AIRPORT_ID', 'DEP_DELAY', 'ARR_DELAY']`
   - Cast `DEP_DELAY` and `ARR_DELAY` as a float. These are minutes that are delayed.
   - Subtract `DEP_DELAY` from `ARR_DELAY` to get the actual `ARR_DELAY`
   - If a flight is `CANCELLED`, add 5 hours to `DEP_DELAY`
   - There are missing values in `DEP_DELAY` and `ARR_DELAY` (i.e. `''`) and 
     you would want to replace those with `0`.
     
   Map `make_rows()` to the RDD and you should have an RDD where each item is 
   a dictionary.
   
8. Instead of dictionaries, make 2 RDDs where the items are tuples.
   The first RDD will contain tuples `(DEST_AIRPORT_ID, ARR_DELAY)`. 
   The other RDD will contain `(ORIGIN_AIRPORT_ID, DEP_DELAY)`.
   Run a `.first()` or `.take()` to confirm your results.

9. Make 2 RDDs for the mean delay time for origin airports and destination
   airports. You will need to `groupByKey()` and then take the mean of the 
   delay times for the particular airport. This is where having our RDDs be 
   composed of `(key, value)` pairs really helps - it allows us to use the 
   `groupByKey()` method on our RDD. 

10. Run `rdd.persist()` on the RDDs you made in `9.`. Remember to set the 
   name of the RDD using `.setName()` before running `persist()` (e.g. 
   `rdd.setName('airline_rdd').persist()`). Setting the name will allow you to 
   identify the RDD in the Spark UI. When you cache the RDDs, you make sure that  
   they do not need to be reproduced every time they are called upon. 
   Use `persist()` for RDDs that you are going to repeatedly use.

11. Use `rdd.sortBy()` to sort the RDDs by the mean delay time to answer the 
    following questions.

    - Top 10 departing airports that have the lowest average delay
    - Top 10 departing airports that have the highest average delay
    - Top 10 arriving airports that have the lowest average delay
    - Top 10 arriving airports that have the highest average delay 

  
[RDDs]: http://spark.apache.org/docs/latest/programming-guide.html#resilient-distributed-datasets-rdds 
[funct-programming]: https://en.wikipedia.org/wiki/Functional_programming
[wiki-lazy-eval]: http://en.wikipedia.org/wiki/Lazy_evaluation
[RDD-docs]: http://spark.apache.org/docs/0.7.3/api/pyspark/pyspark.rdd.RDD-class.html 
[groupby-v-reduceby-key]: https://github.com/databricks/spark-knowledgebase/blob/master/best_practices/prefer_reducebykey_over_groupbykey.md

