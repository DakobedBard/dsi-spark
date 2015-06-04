# Spark Web UI guide

1. In the UI, if you click on **`PySparkShell`** (in the list of applications) You'd be routed to the **Application** page: 

   ![running_application](images/running_application.png)

   ![pysparkshell_page](images/pysparkshell_page.png)

2. Click on the **Application Detail UI** and you see the Spark Jobs page which contains detailed execution information
   for active and recently completed Spark jobs. (We don't see any jobs since we haven't run any jobs yet) As we go 
   through this exercise, we want to monitor our Spark jobs.

3. Loading in the airline data. Note that after this, there would still not be any jobs on the UI showing up 
   due to lazy evaluation.

   ```python
   # DON'T INCLUDE THE '[' AND ']'
   link = s3n://[YOUR_AWS_ACCESS_KEY_ID]:[YOUR_AWS_SECRET_ACCESS_KEY]@mortar-example-data/airline-data
   airline = sc.textFile(link)
   ```

4. Run a `count()` on the airline data which is a Spark job.
   When `count()` is running go back to the Spark Jobs page and refresh.
   You should see the count running as an active job. 

   ![count_job](https://github.com/zipfian/spark/blob/master/images/count_job.png)

5. Click into the Description  









