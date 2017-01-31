## Part 1: Introduction to SparkSQL

SparkSQL allows you to execute relational queries on **structured** data using
Spark. Today we'll get some practice with this by running some queries on a
Yelp dataset. To begin, you will load data into a Spark `DataFrame`, which can
then be queried as a SQL table.

<br>

### 1. Validate your AWS CLI configuration

This exercise requires Spark 2 to be installed on your computer. If you have not yet installed Spark 2, refer to the [installation guidelines](https://github.com/zipfian/spark-install) for instructions.

At your computer's bash terminal, run `aws s3 ls sparkdatasets` to confirm that you have `awscli` installed and configured correctly on your computer. You should see a listing of the contents of the `sparkdatasets` bucket:

```shell
$ aws s3 ls sparkdatasets
                           PRE sotu/
2015-07-11 22:32:06    6588490 RadiusDataEngineeringExercisePySpark.tgz
2015-07-11 22:35:35      37514 donotcall.txt
2015-08-29 19:22:32     575014 ebay.csv
2015-07-29 12:48:18   51118296 mnist_test.csv
2015-07-07 13:03:13   26582277 news.txt
2015-07-07 13:12:24  100000000 text8_lines
2015-07-11 22:35:36   14638824 transactions.txt
2015-07-11 22:35:44    1877331 users.txt
2015-07-07 13:15:51   55412111 yelp_academic_dataset_business.json
2015-07-07 13:16:16   20664236 yelp_academic_dataset_checkin.json
2015-07-10 15:25:48 1426365176 yelp_academic_dataset_review.json
2015-07-07 13:16:30  166192435 yelp_academic_dataset_user.json
```

If you receive an error message, please ensure that you have `awscli` installed (`pip install awscli`) and configured (`aws configure`) before proceeding with the exercise. If your `AWS_SECRET_ACCESS_KEY` contains one or more slashes, you may need to log into the AWS console and generate a new key that does not contain slashes.

### 2. Start a PySpark-enabled Jupyter Notebook server.

Run `bash scripts/jupyspark.sh`. This script starts a PySpark-enabled Jupyter Notebook server that with a fully initialized SparkContext called `sc` and a SparkSession called `spark`.

### 3. Load data into a Spark DataFrame.

1\. Use the s3a filesystem type to load the dataset using the same stored
  credentials used by the AWS CLI client. This eliminates the need to
  manage your AWS access keys manually in your Python code.

   ```python
   yelp_business_url = 's3a://sparkdatasets/yelp_academic_dataset_business.json'
   yelp_business_df = spark.read.json(yelp_business_url)
   ```

   This 53-megabyte file will take a moment to load. You can monitor the progress of the job by opening your local PySparkShell UI in a separate browser tab: [http://localhost:4040/stages/](http://localhost:4040/stages/)

   Note: if you have not configured your AWS CLI correctly, this will not work for you. An alternate approach for reading this file is to [download it locally](https://s3.amazonaws.com/sparkdatasets/yelp_academic_dataset_business.json) and use the the `.read.json()` method on the local file. However, this approach is NOT recommended because it does not scale to larger datasets.

2\. Print the schema and register the `yelp_business_df` as a temporary
table named `yelp_business` (this will enable us to query the table later using
our `SparkSession()` object).

    ```python
    yelp_business_df.printSchema()
    yelp_business_df.registerTempTable("yelp_business")
    ```

    Now, you can run SQL queries on the `yelp_business` table. For example:

    ```python
    result = spark.sql("SELECT name, city, state, stars FROM yelp_business LIMIT 10")
    result.show()
    ```

3\. Write a query or a sequence of transformations that returns the `name` of entries that fulfill the following
conditions:

- Rated at 5 `stars`
- In the `city` of Phoenix
- Accepts credit card (Reference the `'Accepts Credit Card'` field by
``` attributes.`Accepts Credit Cards` ```)
- Contains Restaurants in the `categories` array.  

Hint: `LATERAL VIEW explode()` can be used to access the individual elements
of an array (i.e. the `categories` array). For reference, you can see the
[first example](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LateralView) on this page.

Hint: In spark, while using `filter()` or `where()`, you can create a condition that tests if a column, made of an array, contains a given value. The functions is [pyspark.sql.functions.array_contains](http://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.functions.array_contains).


<br>

## Part 2: Spark and SparkSQL in Practice

Now that we have a basic knowledge of how SparkSQL works, let's try dealing with a real-life scenario where some data manipulation/cleaning is required before we can query the data with SparkSQL. We will be using a dataset of user information and a data set of purchases that our users have made. We'll be cleaning the data in a regular Spark RDD before querying it with SparkSQL.

1\. Load a dataframe `users` from S3 link `''s3a://sparkdatasets/users.txt'` (no credentials needed) using `spark.read.csv` with the following parameters: no headers, use separator `";"`, and infer the schema of the underlying data (for now). Use `.show(5)` and `.printSchema()` to check the result.

2\. Create a schema for this dataset using proper names and types for the columns, using types from the `pyspark.sql.types` module (see lecture). Use that schema to read the `users` dataframe again and use `.printSchema()` to check the result.

Note: Each row in the `users` file represents the user with his/her `user_id, name, email, phone`.

3\. Load an RDD `transactions_rdd` from S3 link `''s3a://sparkdatasets/transactions.txt'` (no credentials needed) using `spark.sparkContext.textFile`. Use `.take(5)` to check the result.

Use `.map()` to split those csv-like lines, to strip the dollar sign on the second column, and to cast each column to its proper type.

4\. Create a schema for this dataset using proper names and types for the columns, using types from the `pyspark.sql.types` module (see lecture). Use that schema to convert `transactions_rdd` into a dataframe `transactions`  and use `.show(5)` and `.printSchema()` to check the result.

Each row in the `transactions` file has the columns  `user_id, amount_paid, date`.

5\. Write a sequence of transformations or a SQL query that returns the names and the amount paid for the users with the **top 10** transaction amounts.

[lateral-view]: https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LateralView
