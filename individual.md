##Part 1: RDD and Spark Basics

Here we will get familiar with the basics of Spark (PySpark). We will be just using a master/driver node.
In `Part 3`, we are going simulate a master-worker cluster to run our jobs.

<br>

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
   
   Read an RDD in from a text file. 
   **By default, the RDD will treat each line as an item and read it in as string.**
   
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
<br>

##Part 2: Intro to Functional Programming

Spark operations conforms to the functional programming paradigm. Objects (RDDs) are immutable 
and mapping a function to an RDD returns another RDD. A lot of Spark's functionalities assume the 
items in an RDD to be tuples of `(key, value)`. Structure your RDDs to be `(key, value)` whenever possible.

Also beware of [**lazy evaluation**](http://en.wikipedia.org/wiki/Lazy_evaluation) where operations
are not executed until a `.collect()`, `.first()`, `.take()` or `.count()` is call to retrieve items
in the RDD.

**If you are not sure what RDD transformations/actions there are, check out [http://spark.apache.org/docs/0.7.3/api/pyspark/pyspark.rdd.RDD-class.html](http://spark.apache.org/docs/0.7.3/api/pyspark/pyspark.rdd.RDD-class.html)**

<br>

1. Turn the items in `file_rdd` into `(key, value)` pairs using `map()` and a `lambda` function. Map each item into    a json object (use `json.loads`) and then map to the `(key, value)` pairs. **Remember to cast value as type**  
   `int`.  Use `collect()` to see your results. Using `collect()` is fine here since the data is small.
   
   - **The key is the name of the person**
   - **The value is how many chocolate chip cookies bought**
    
2. Now use `filter()` to look for entries with more than `5` chocolate chip cookies.

3. For each name, return the entry with the max number of cookies. 
   
   **Hint:** 
   - Use `reduceByKey()` instead of `groupByKey()`. See why [here](https://github.com/databricks/spark-knowledgebase/blob/master/best_practices/prefer_reducebykey_over_groupbykey.md)
   - You will see a `pyspark.resultiterable.ResultIterable` returned. It has a `.data` attribute 
     which you can use to access the Python list
 
4. Calculate the total revenue from people buying cookies.

   **Hint:**
   - `rdd.values()` returns another RDD of all the values
   - Use `reduce()` to return the sum of all the values

<br>
   
##Part 3: Starting a Local Cluster

Here we will simulate starting a master/worker cluster locally. That allows us to develop code on a local cluster
before deployment. We will be using [`tmux`](http://tmux.sourceforge.net/) to run our scripts in the background .
`tmux` lets us *multiplex* your terminal, create terminal sessions, and attach/detach 
different programs in the terminal (somewhat like running processes in hidden terminals). **Below is just a quick
guide to tmux for you to skim through.**

<br>

1. To get tmux run:
   
   ```bash
   brew install tmux
   ```

2. To start a new tmux session run:
   ```bash
   tmux new -s [session_name]
   ```

3. To detach a tmux session use:
   ```bash
   ctrl+b, d
   ```

4. To get a list of your current tmux sessions run:
   ```bash
   tmux ls
   ```

5. To attach an existing session run:
   ```bash
   tmux attach -t [session_name]
   ```

<br>

Now we can use `tmux` to create a local cluster (master and workers) which will be running in terminals in the background. 

<br>

1. Start a tmux session which will host your master node:

   ```bash
   tmux new -s master
   ```
2. Run the following command to set up the Spark master to listen on local IP. The Master class in 
   `org.apache.spark.deploy.master` accepts the following parameters
   
   - `h` : host (which on our case is local host `127.0.0.1`) 
   - `p`: The port on which the master is listening in (`7077`)
   - `webui-port`: The port on which the webui is reachable (`8080`)
   
   <br>

   ```bash
   ${SPARK_HOME}/bin/spark-class org.apache.spark.deploy.master.Master\
   -h 127.0.0.1\
   -p 7077\
   --webui-port 8080
   ```
3. You should get some output in your terminal similar to the following:
   ![master_term](https://github.com/zipfian/spark/blob/master/images/master_term.png)

4. Detach from your master session(`crtl+b, d`). Start a new tmux session:
   
   ```bash
   tmux new -s worker1
   ```

5. Start a worker by running the following:

   ```bash
   ${SPARK_HOME}/bin/spark-class org.apache.spark.deploy.worker.Worker \
   -c 1 \
   -m 1G \
   spark://127.0.0.1:7077 \
   ```
   
   This will start a worker with 1GB memory and 1 core and attach it to the previously created Spark master. 
   The output in your terminal should be:
   
   ![worker_term](https://github.com/zipfian/spark/blob/master/images/worker_term.png)

   Detach the current session and create a new session and run the same command to create a second worker. 

6. You have set up a master with 2 workers locally. Spark also provides us with a web UI that lets us track the Spark
   jobs and see other stats about any Spark related tasks and workers. 

   **Your web UI is at: `localhost:8080`**

   ![sparkui_first](https://github.com/zipfian/spark/blob/master/images/sparkui_first.png)

7. We are not running any applications with our local Spark cluster yet. We can attach an IPython notebook to the 
   master and start `pyspark` by running the following command which starts the notebook in the browser
   and assigns 1G of RAM per executor and 1G of RAM to the master in pyspark application.
   
   ```bash
   IPYTHON_OPTS="notebook"  ${SPARK_HOME}/bin/pyspark \
   --master spark://127.0.0.1:7077 \
   --executor-memory 1G \
   --driver-memory 1G
   ```

8. A SparkContext is already loaded into IPython. Access the SparkContext with the variable `sc`.

   You will see an output like below:

   ```python
   sc
   pyspark.context.SparkContext at 0x104318250
   ```

9. Now if you refresh your spark web UI, you should see **`PySparkShell`** running in the list of applications. 
   
  ![running_application](https://github.com/zipfian/spark/blob/master/images/running_application.png)

 
<br>

##Part 4: Spark for Data Processing 

Using the cluster we have set up in `Part 3`, we will be dealing with airport data and we would want to identify
airports with the worst / least delay.
 
**2 types of delays:**

- Arrival delays (`ARR_DELAY`) and departure delays (`DEP_DELAY`)
- All delays are in terms of **minutes**
- `ARR_DELAY` is associated with the destination airport (`DEST_AIRPORT_ID`)
- `DEP_DELAY` is associated with the destination airport (`ORIGIN_AIRPORT_ID`)

<br>

1. Read [sparkui.md](sparkui.md) for further guide as to how to use the UI. The guide will bring you through
   `2.` and `3.`.

2. Load the file in as follow.

   ```python
   # DON'T INCLUDE THE '[' AND ']'
   link = 's3n://[YOUR_AWS_ACCESS_KEY_ID]:[YOUR_AWS_SECRET_ACCESS_KEY]@mortar-example-data/airline-data'
   airline = sc.textFile(link)
   ```
   
2. Print the first 2 entries. The first line is the column names and starting from the second line is 
   the corresponding data. Also run a `.count()` on the RDD. This will **take a while** as the data set is
   a few million rows. 

3. As you can see `.count()` takes a long time to run. It's a common practice to sub-sample your data when writing
   your code so you don't have to wait for different commands to run. You can use `.take(100)` to sample out the
   first 100 rows and assign it to a new RDD using `sc.parallelize`.

4. Let's do some preprocessing. Remove the `'`, `"` and the trailing `,` for each line. Print the first 2 lines
   to confirm. The first 2 lines should look like the following.
   
   ```
   YEAR,MONTH,UNIQUE_CARRIER,ORIGIN_AIRPORT_ID,DEST_AIRPORT_ID,DEP_DELAY,DEP_DELAY_NEW,ARR_DELAY,ARR_DELAY_NEW,CANCELLED
   2012,4,AA,12478,12892,-4.00,0.00,-21.00,0.00,0.00
   ```
  
5. Use `filter` to filter out the line containing the column names. 


6. Make a function, `make_rows()`, that takes a line as an argument and return a dictionary
   where the keys are the column names and the values are the values for the column. 
   
   - The output is a dictionary with only these columns:
     `['DEST_AIRPORT_ID', 'ORIGIN_AIRPORT_ID', 'DEP_DELAY', 'ARR_DELAY']`
   - Cast `DEP_DELAY` and `ARR_DELAY` as a float. These are minutes that are delayed.
   - Subtract `DEP_DELAY` from `ARR_DELAY` to get the actual `ARR_DELYAY`
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

10. Use `rdd.sortBy()` to sort the RDDs by the mean delay time to answer the following questions.

    - Top 10 departing airport that has least avgerage delay in minutes
    - Top 10 departing airport that has most avgerage delay in minutes
    - Top 10 arriving airport that has least avgerage delay in minutes
    - Top 10 arriving airport that has most avgerage delay in minutes

   

