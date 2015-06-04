##Miniquiz

1. You are a data scientist at an advertisement company and you are interested if the new campaign is boosting
   sales relative to the previous campaign. Describe the steps required to establish that the new campaign **causes**
   sales to go up or down.

2. What is a p-value and statistical power? What is the relationship between the two. Feel free to look up if you 
   need to.

##Spark / PySpark Installation

If you want to install PySpark on your laptop

1. Go to this [link](http://spark.apache.org/downloads.html). 
2. Select `Pre-built for Hadoop 1.X` under `2. Chose a package type:`.
3. Download the tar package by `Download Spark: spark-1.2.1-bin-hadoop1.tgz`. Version `1.3` would work too.
4. Unzip the file and place it at your home directory
5. Make sure the folder name is `spark-1.2.1-bin-hadoop1`
6. Include the following lines in the `~/.bashrc` or `~/.zshrc` file

   ```
   export SPARK_HOME=[path to your unzipped spark folder]
   export PYTHONPATH=[path to your unzipped spark folder]python/:$PYTHONPATH
   ```
7. Also install py4j by `sudo pip install py4j`

8. Now open up an ipython terminal and `import pyspark as ps`. If it did not throw an error,
   then you are ready to go.
   

