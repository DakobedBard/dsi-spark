## Part 1: Introduction to SparkSQL

SparkSQL allows you to execute relational queries on **structured** data using
Spark. Today we'll get some practice with this by running some queries on a
Yelp dataset. To begin, you will load data into a Spark `DataFrame`, which can
then be queried as a SQL table.

<br>

### 1. Validate your AWS CLI configuration

This exercise requires Spark 2 to be installed on your computer. If you have not yet installed Spark 2, refer to the [install_spark miniquiz](https://github.com/zipfian/miniquizzes/blob/master/install_spark.md) for instructions.

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

  Use the s3a filesystem type to load the dataset using the same stored
  credentials used by the AWS CLI client. This eliminates the need to
  manage your AWS access keys manually in your Python code.

   ```python
   yelp_business_url = 's3a://sparkdatasets/yelp_academic_dataset_business.json'
   yelp_business_df = spark.read.json(yelp_business_url)
   ```

   This 53-megabyte file will take a moment to load. You can monitor the progress of the job by opening your local PySparkShell UI in a separate browser tab: [http://localhost:4040/stages/](http://localhost:4040/stages/)

   Note: if you have not configured your AWS CLI correctly, this will not work for you. An alternate approach for reading this file is to [download it locally](https://s3.amazonaws.com/sparkdatasets/yelp_academic_dataset_business.json) and use the the `.read.json()` method on the local file. However, this approach is NOT recommended because it does not scale to larger datasets.

2. Print the schema and register the `yelp_business_df` as a temporary
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

3. Write a query that returns the `name` of entries that fulfill the following
conditions:

   - Rated at 5 `stars`
   - In the `city` of Phoenix
   - Accepts credit card (Reference the `Accepts Credit Card` field by
   ````attributes.`Accepts Credit Cards`````)
   - Contains Restaurants in the `categories` array.  

   Hint: `LATERAL VIEW explode()` can be used to access the individual elements
   of an array (i.e. the `categories` array). For reference, you can see the
   [first example][lateral-view] on this page.

<br>

## Part 2: Spark and SparkSQL in Practice

Now that we have a basic knowledge of how SparkSQL works, let's try dealing with a real-life scenario where some data manipulation/cleaning is required before we can query the data with SparkSQL. We will be using a dataset of user information and a data set of purchases that our users have made. We'll be cleaning the data in a regular Spark RDD before querying it with SparkSQL.

1. Load the `user` and `transaction` datasets into 2 separate RDDs with the
following code.

   ```python
   user_rdd = sc.textFile('s3a://sparkdatasets/users.txt')
   transaction_rdd = sc.textFile('s3a://sparkdatasets/transactions.txt')
   ```

2. Each row in the `user` RDD represents the user with his/her `user_id, name, email, phone`. Each row in the `transactions` RDD has the columns  `user_id, amount_paid, date`. Map a function to the `user` RDD to make each row in the RDD a json **string** of the form `{user_id: XXX, name: XXX, email:XXX, phone:XXX}` (use `json.dumps()`). Map a function to the `transactions` RDD to make each row a json **string** of the form `{user_id: XXX, amount_paid: XXX.XX, date: XXX}`.  

   **Hint: Strip the `$` sign in the `amount_paid` column in the `transactions`
   RDD so it will be recognized as a float when read into a Spark DataFrame.**

3. Convert the `user` and `transactions` RDDs to Spark DataFrames (use the
   `.read.json()` method on your `SparkSession` object). Print the schemas
   to make sure the conversion is successful. Register the Spark DataFrames as
   separate tables and print the first couple of rows with SQL queries.

4. Write a SQL query to return the names and the amount paid for the users with
   the **top 10** transaction amounts.

[lateral-view]: https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LateralView
