##Part 1: Machine Learning in Spark

Here we are going to run some Machine Learning Algorithms using MLlib, the Machine Learning library for Spark.
We are going to build a Naive Bayes model to predict the category of the newsgroup article based on it content.

1. Import these libraries

   ```python
   import string
   import json 
   import pickle as pkl
   from pyspark.mllib.feature import HashingTF
   from pyspark.mllib.regression import LabeledPoint
   from pyspark.mllib.classification import NaiveBayes
   from collectons import Counter
   ```

2. Start a local cluster just as you have done this morning. Load the text file into an RDD. Map the lines
   to dictionaries. Take the first 2 lines to confirm your results.

   ```python
   data_raw = sc.textFile('s3n://[YOUR_AWS_ACCESS_KEY_ID]:[YOUR_AWS_SECRET_ACCESS_KEY]@newsgroup/news.txt')
   ```
   
3. Check how many partitions are in you RDD by `getNumPartitions`. If it is too low (i.e. 2), you might want
   to repartition your RDD with [`.repartition()`](https://spark.apache.org/docs/1.1.1/api/python/pyspark.rdd.RDD-class.html#repartition)
   to a higher number of partition. Whilst partitioning beyond the number of your core does not in theory 
   increase parallelism, it allows you to track your tasks within a job better (since smaller tasks).
   
4. There are 3 fields in the data: `label`, `label_name` and `text`.

   - Make an RDD of unique `(label, label_name)` pairs. This creates a reference 
     of the numeric label to the name of the label. We will need it later.
   
   - Make another RDD with `(label, text)`. We will make bag-of-words 
     and compute term frequency out of the `text`. This will be the 
     target and feature we will later train our Navie Bayes on.

5. Tokenize your text by mapping the following functions: 
   
   Feel free to use `nltk`. Also feel free to wrap all of the functions in one big function and then map.

   - Lower casing the text
   - Removing puntuations
   - Removing stop words
   - Splitting on white spaces
   - Stem the tokenized words
   
6. You should now have a `(key, value)` pair RDD where the `key` is the numeric value that 
   represents the category of newsgroup of the article and the `value` is a list of stemmed 
   words of the article.
   
   Create a new RDD where the key remain to be the category and value is the term-frequency (TF) vector for
   each article. This is going to require some thinking.
    
   You may not use the built-in `HashingTF()`, nor use `collect()` on the whole corpus. You are only allowed to 
   do a `collect()` of an RDD that contains the vocab of the corpus.
   
   Here are some functions you might want to consider (not necessarily in order) to implement TF.
   
   - `.values()`
   - `.flatMap()`
   - `.distinct()`
   - `.mapValues()`
   - `Counter()` (Python function)
   
7. Use `HashingTF()` to get the word vector in order to confirm your implementation is correct. Your first row should
   have a sum of 186 words. You cannot compare the actual word vec since `HashingTF()` would have mixed up the ordering
   of the vocab.
   
      ```python
   ### Using HashingTF() ###
   htf = HashingTF(10000)
   word_vecs_rdd = words_rdd.mapValues(htf.transform)
   word_vec_rdd.count()
   ```
   
   One advantage with your implementation is that you have access to the actual vocab itself where `HashingTF()` would 
   have obscured the words by hashing them.

8. Spark uses a `LabeledPoint(target, feature)` object to store the target (numeric)
   and features (numeric vector) for all machine learning algorithms. 
   Make a new RDD by mapping `LabeledPoint(target, feature)` to the RDD and keep the new RDD in 
   cache using `persist()`. Remember to `setName()` before you persist. The `feature` is the TF vector.
   
9. Use `randomSplit()` (RDD built-in method) to do a train test split of `70:30`.

10. Train the `NaiveBayes` model on the train data set. See the 
   [docs](http://spark.apache.org/docs/1.2.0/api/python/pyspark.mllib.html#module-pyspark.mllib.classification)
   here.
   
11. Map the `predict()` function of the `NaiveBayes` model onto the test set features to get
   predictions. Caculate the accuracy of the predictions.

12. Examine the predictions that are incorrect. Use the dictionary you have created in `3.` to get
    the `label_name` of those predictions. Examine the content of the incorrect predictions and
    try to reason why the content is incorrectly predicted.

13. Save your model as a pickle file. This will allow you to use the model make prediction about the 
    label of new articles.

14. To make prediction of new data, write a standalone python script that would take the file name
    of the file containing the new articles as a command line argument (use `argparse`). 
    
    At this point, you are just write Python and has nothing to do with Spark. This is just good Python practice 
    for you.
    
    The script should do the following:
    - Read and preprocess the new data (`data/news_test.pkl`)
    - Preprocessing must be the same as you have done to build the model above
    - Put the new data in an RDD
    - Find the 20 most common words in each article
    - Unpickle your model and make predictions for the label of the article
    - Write out a file with the predicted labels and the most common words 
    