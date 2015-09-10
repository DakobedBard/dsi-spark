##Miniquiz

1. You are a data scientist at an advertisement company and you are
   interested if the new campaign is boosting sales relative to the
   previous campaign. Describe the steps required to establish that
   the new campaign **causes** sales to go up or down.

2. What is a p-value and statistical power? What is the relationship
   between the two. Feel free to look up if you need to.

##Spark / PySpark Installation

If you want to install PySpark on your laptop

1. Go to this [link](http://spark.apache.org/downloads.html). 

2. For *Choose a Spark release:* select *1.4.1 (Jul 15 2015)*

3. For *Choose a package type:* select *Pre-built for Hadoop 2.4 and later*

   ![](images/spark-download-choices.png)

4. Download the tgz package.

5. Do **not** download the latest version. It has bugs we will talk about.

6. Make sure you are downloading the binary version, not the source
   version.

7. Unzip the file and place it at your home directory

8. Include the following lines in the `~/.bash_profile` on Mac (without the brackets).

   ```
   export SPARK_HOME=[full path to your unzipped spark folder]
   export PYTHONPATH=[full path to your unzipped spark folder]/python/:$PYTHONPATH
   ```
9. Install py4j using `sudo pip install py4j`

10. Open a new terminal window.

11. Start ipython console and type `import pyspark as ps`. If it did
    not throw an error, then you are ready to go.

12. Start `ipython notebook` from the new terminal window.

13. You might have download the newest version of
    [`JDK`](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
    if PySpark is throwing errors about Java.
