# Spark Web UI guide

We usually run Spark on a cluster of machines with a master node and a number of worker nodes, but during development and for testing purposes we want to run spark locally and once we have everything right, then we deploy Spark to a cluster.  
Here, we want to set one Master node and 2 Workers locally by distributing memory and processor resources to them locally and mimic the cluster environment.
To do this, we will need to run some commands to create our master and worker nodes.

## Quick intro to tmux
We will be using [`tmux`](http://tmux.sourceforge.net/) to run our scripts in the background. `tmux` lets us *multiplex* your terminal, create terminal sessions, and attach/detach different programs in the terminal.
To get tmux run:
```shell
brew install tmux
```

To start a new tmux session run:
```shell
tmux new -s *session_name*
```

To detach a tmux session use:
```shell
ctrl+b, d
```

To get a list of your current tmux sessions run:
```shell
tmux ls
```

To attach an existing session run:
```shell
tmux attach -t *session_name*
```

## Creating Spark master and worker with tmux
Start a tmux session:
```shell
tmux new -s master
```
and run the following command to set up the Spark master to listen on local IP. The Master class in org.apache.spark.deploy.master accepts the following parameters
	* h : host (which on our case is local host 127.0.0.1) 
	* p: The port on which the master is listening in (7077)
	* webui-port: The port on which the webui is reachable (8080)
```shell
${SPARK_HOME}/bin/spark-class org.apache.spark.deploy.master.Master -h 127.0.0.1 -p 7077 --webui-port 8080
```
You should get some output in your terminal similar to the following:
[master_term](ADD LINK)

## Creating 2 Spark workers with tmux
Detach from your master session(`crtl+b, d`). Start a new tmux session:
```shell
tmux new -s worker1
```
Start a worker by running the following:
```shell
${SPARK_HOME}/bin/spark-class org.apache.spark.deploy.worker.Worker spark://127.0.0.1:7077 -c 1 -m 1G
```
This will start a worker with 1GB memory and 1 core and attach it to the previously created Spark master. The output in your terminal should be:
[worker_term](ADD LINK)
Detach the current session and create a new session and run the same command to create a second worker. 

You have set up a master with 2 workers locally. Spark also provides us with a web UI that lets us track the Spark jobs and see other stats about any Spark related tasks and workers. 
Your web UI is at: `localhost:8080`
[sparkui_first](ADD LINK)

We are not running any applications with our local Spark cluster yet. We can attach an Ipython notebook to the master and start `pyspark` by running the following command which starts the notebook in the browser and assigns 1G of RAM per executor to the pyspark application.
```shell
IPYTHON_OPTS="notebook"  ${SPARK_HOME}/bin/pyspark --master spark://127.0.0.1:7077 --executor-memory 1G --driver-memory 1G
```
Now if you refresh your spark web UI, you should see PySparkShell running in the list of applications. 
[running_application](ADD LINK)

A SparkContext is already loaded when PySparkShell is running. Access the SparkContext with the variable `sc`.
You will see an output like below:
```python
sc
pyspark.context.SparkContext at 0x104318250
```

In the UI, if you click on PySparkShell (in the list of applications) You'd be routed to the following page: 

[pysparkshell_page](ADD LINK)

Click on the **Application Detail UI** and you see the Spark Jobs page which contains detailed execution information for active and recently completed Spark jobs. (We don't see any jobs since we haven't run any jobs yet) As we go through this exercise, we want to monitor our Spark jobs.

Try loading in the airline data we use for part 2 of the morning exercise:
```python
airline = sc.textFile('[AWS_ACCESS_KEY_ID]:[AWS_SECRET_ACCESS_KEY]@s3n://mortar-example-data/airline-data')
```

Run a `count()` on the airline data which is a Spark job. This will take a while as the data set is a few million rows. 
When `count()` is running go back to the Spark Jobs page and refresh.
You should see the count running as an active job. 

[count_job](ADD LINK)










