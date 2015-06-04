# Spark Web UI guide

In the UI, if you click on **`PySparkShell`** (in the list of applications) You'd be routed to the following page: 

![running_application](images/running_application.png)


![pysparkshell_page](images/pysparkshell_page.png)

Click on the **Application Detail UI** and you see the Spark Jobs page which contains detailed execution information for active and recently completed Spark jobs. (We don't see any jobs since we haven't run any jobs yet) As we go through this exercise, we want to monitor our Spark jobs.

Try loading in the airline data we use for part 2 of the morning exercise:
```python
airline = sc.textFile('[AWS_ACCESS_KEY_ID]:[AWS_SECRET_ACCESS_KEY]@s3n://mortar-example-data/airline-data')
```

Run a `count()` on the airline data which is a Spark job. This will take a while as the data set is a few million rows. 
When `count()` is running go back to the Spark Jobs page and refresh.
You should see the count running as an active job. 

![count_job](https://github.com/zipfian/spark/blob/master/images/count_job.png)










