# Part 1: Machine Learning in Spark

Here we are going to run some Machine Learning Algorithms using the ML pipeline. We are going to train a Naive Bayes model to discriminate positive and negative reviews based on text.

We recommend you to work from a notebook using [miniquizz/install_spark](https://github.com/zipfian/miniquizzes/blob/master/install_spark.md) instructions.

## 1.1. Load data from Amazon Reviews json file in a snap

First, we will work on a local json datafile which contains a limited subset of the Amazon Reviews.

1\. Use `sqlContext.read.json()` to create a dataframe that would contain the content of the file `'data/reviews_Musical_Instruments_5.json.gz'`. Check the structure of that dataframe, and the column detected in the json content, by using `.printSchema()`. It should read like :

```
root
 |-- asin: string (nullable = true)
 |-- helpful: array (nullable = true)
 |    |-- element: long (containsNull = true)
 |-- overall: double (nullable = true)
 |-- reviewText: string (nullable = true)
 |-- reviewTime: string (nullable = true)
 |-- reviewerID: string (nullable = true)
 |-- reviewerName: string (nullable = true)
 |-- summary: string (nullable = true)
 |-- unixReviewTime: long (nullable = true)
 ```

 3\. From now on, we will keep only the columns `reviewText` and `overall`. Use `.select()` on the dataframe to keep those two only. You can check your transformation using `.printSchema()` again.

## 1.2. Create a `label` for classification and a balanced dataset

 This dataset is made of user reviews and ratings:

* the `reviewText` column (string) contains the raw text of the review.
* the `overall` column (double) contains the rating given by the user, in `{1.0, 2.0, 3.0, 4.0, 5.0}`

1\. Using `.groupBy()` and `.agg()`, count the number of reviews in each rating value.

2\. We are going to focus on extreme ratings `{1.0, 5.0}`. Using your count, identify how much examples in each of these two classes we need to keep to build a balanced set of examples having the same number of reviews in `1.0` and `5.0`.

3\. By using `.filter()` on your dataframe create two dataframes:

* one for the reviews having an `overall` of `1.0` (we will call them the `neg` class),
* another for the reviews having an `overall` of `5.0` (we will call them the `pos` class).

Limit the number of reviews in each dataframe by the number you have identified previously. Be sure to shuffle those reviews before you apply your limit (you can use `.orderBy(rand())` for that).

Using `.union()` between dataframes, create a single dataframe containing the samples from both the balanced `neg` and `pos` classes.

4\. Using `.withColumn()` create a new column called `label` that has a value of `0.0` for the `neg` class, and `1.0` for the `pos` class.

5\. Check your dataframe at this step using `.printSchema()`. It should look like:

```
root
 |-- reviewText: string (nullable = true)
 |-- overall: double (nullable = true)
 |-- label: double (nullable = true)
```

## 1.3. Build a step by step text indexation pipeline

We are now going to index each reviews by its `reviewText` using an nlp indexation pipeline.

1\. Create a function `preprocess_raw_text()` that takes a string as an input, and outputs the tokens of the text using the following operations :

* tokenizing the text using punctuation
* removing stopwords
* stemming each word to reduce it to its root

You should use the `nltk` library or the `pattern` library to do that.

**Recommendation**: use any paragraph of (real) raw text to test your function before going on the next step.

2\. In order to apply a function to the values in an RDD, you used `.map()` with a lambda function, or your own. DataFrames let you do that within a `.withColumn()`, using a container called `udf` for User Defined Function (it's in the `pyspark.sql.functions` module).

For a function of your own design, you need to create a `udf` by specifying the type of the output valuesof your function. This is required so that the dataframe knows what type the new column will be in the dataframe schema. For your `preprocess_raw_text()` function, the return type would be `ArrayType(StringType())` (from the `pyspark.sql.types` module). This would look like :

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import ArrayType, StringType

tokenizer_udf = udf(lambda x : preprocess_raw_text(x), ArrayType(StringType()))
```

You can now apply this `tokenizer_udf` function in a `.withColumn()` transformation to create a column named `'tokens'` containing the list of tokens extracted from the `'reviewText'` column.

At this step, `.printSchema()` should read :
```
root
 |-- reviewText: string (nullable = true)
 |-- overall: double (nullable = true)
 |-- label: double (nullable = true)
 |-- tokens: array (nullable = true)
 |    |-- element: string (containsNull = true)
```

3\. The basics of ML pipelining in spark relies on building step by step instances of classes drawn from the `pyspark.ml` library. Among these classes we are going to use the following :

* [`pyspark.ml.feature.CountVectorizer`](http://spark.apache.org/docs/latest/api/python/pyspark.ml.html#pyspark.ml.feature.CountVectorizer) : computes term frequency vertors from a lists of tokens
* [`pyspark.ml.feature.IDF`](http://spark.apache.org/docs/latest/api/python/pyspark.ml.html#pyspark.ml.feature.IDF) : computes inverse document frequences on term frequency vectors
* [`pyspark.ml.classification.NaiveBayes`](http://spark.apache.org/docs/latest/api/python/pyspark.ml.html#pyspark.ml.classification.NaiveBayes) : implements the Naive Bayes algorithm

The syntax is usually the same for these classes :

1. you create an instance `i` specifying input and output columns, plus necessary keyword arguments specific to the class
2. you fit the instance `i` on your dataframe with a `i.fit()` method, fitting will return a model `m`
3. you apply the model `m` on your dataframe using `m.transform()`

As an example for the `IDF` class, `i.fit()` computes the global document frequencies over term the frequency vectors, then `m.transform()` applies the inverse document frequencies over all vectors.

Look for the definitions of these classes in the spark documentation. For each class you should :

* identify the arguments (input/output + parameters) you need to provide.
* identify the right values for these arguments.

As a starter, the following table gives you the input/output arguments for each class.

| Class | input column(s) argument | output column argument | keyword arguments... |
|-------|------------|-------------|----------------------|
| [`CountVectorizer`](http://spark.apache.org/docs/latest/api/python/pyspark.ml.html#pyspark.ml.feature.CountVectorizer) | `inputCol` | `outputCol` | ? |
| [`IDF`](http://spark.apache.org/docs/latest/api/python/pyspark.ml.html#pyspark.ml.feature.IDF) | `inputCol` | `outputCol` | ? |
| [`NaiveBayes`](http://spark.apache.org/docs/latest/api/python/pyspark.ml.html#pyspark.ml.classification.NaiveBayes) | `featuresCol` + `labelCol` | `predictionCol` | ? |


4\. Implement the `CountVectorizer` using your `inputCol` and `outputCol` and keyword arguments. In the returned model, you can access the vocabulary by accessing the internal list `model.vocabulary`. This list is ordered identically as the index of the vectors created by the model. So if you need to find which word correspond to which index of the vector, you can use this list as a reverse table.

5\. Implement the `IDF` using your `inputCol` and `outputCol` and keyword arguments.

At the end of this process, use `.printSchema()` to verify your schema. It should read like this :

```
root
 |-- reviewText: string (nullable = true)
 |-- overall: double (nullable = true)
 |-- label: double (nullable = true)
 |-- tokens: array (nullable = true)
 |    |-- element: string (containsNull = true)
 |-- ******1******: vector (nullable = true)
 |-- ******2******: vector (nullable = true)
```

(replace `*****1*****` above by the output column for `CountVectorizer` and `*****2*****` by the output column for `IDF`).

6\. Before applying the `NaiveBayes` algorithm we will split our dataset into one training set and one testing set using a random split of 70/30. Use [`.randomSplit()`](http://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.DataFrame.randomSplit) to create two distinct dataframes for each of those sets.

**Note**: You can use `.persist()` to create a persistent training set before applying `NaiveBayes`.

7\. Implement `NaiveBayes` specifying the columns for features (`featureCol`), labels (`labelCol`) and prediction (`predictionCol`). Then `.fit()` to obtain a model, and apply this model on the testing test.

8\. Use the [`MulticlassClassificationEvaluator`](http://spark.apache.org/docs/latest/api/python/pyspark.ml.html#pyspark.ml.evaluation.MulticlassClassificationEvaluator) to obtain an evaluation of the accuracy of your classification.

As any other brick in your pipeline, `MulticlassClassificationEvaluator` needs to have columns specified, and some other arguments you need to identify in the documentation. Then, you will need to apply your instance on the prediction and label columns, by using `.evaluate()`. It will compute accuracy (or any other given metric) based on the differences observed between these two columns.

9\. The `NaiveBayes` model provides an internal matrix `model.theta` that you can convert into a numpy array with `model.theta.toArray()`. Use this numpy array to obtain words that are related to the `pos` class, and words that are related to the `neg` class. **Hint**: use `CountVectorizer`'s output vocabulary for that (see question 4).
