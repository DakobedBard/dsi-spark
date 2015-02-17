##1. Machine Learning in Spark

Machine learning at scale becomes easy with Spark. Here we use
Naive Bayes on Term-Frequencies to predict the class of an article
based on its content.

1. Import these libraries

   ```python
   import pyspark as ps
   import string
   import json 
   import pickle as pkl
   from pyspark.mllib.feature import HashingTF
   from pyspark.mllib.regression import LabeledPoint
   from pyspark.mllib.classification import NaiveBayes
   from pyspark.accumulators import AccumulatorParam
   from collectons import Counter
   ```

2. Make a `SparkContext()` and load the text file into an RDD. Map the lines
   to dictionaries. Take the first 2 lines to confirm your results.

   ```python
   data_raw = sc.textFile('s3n://newsgroup/news.txt')
   ```
   
3. There are 3 fields in the data: `label`, `label_name` and `text`.

   - Make an RDD of unique `(label, label_name)` pairs. This creates a reference 
     of the numeric label to the name of the label. We will need it later.
   
   - Make another RDD with `(label, text)`. We will make bag-of-words 
     and compute term frequency out of the `text`. This will be the 
     target and feature we will later train our Navie Bayes on.

4. Tokenize your text by mapping the following functions:

   - Lower casing the text
   - Removing puntuations
   - Removing stop words
   - Splitting on white spaces
   
   **Note:** 
   - You can try using `nltk`, but there are known bugs of using nltk 
     with pyspark. In this case, we will skip over stemming the tokens. 
 
5. Transform the tokenized text to term-frequency (TF) vectors. Consider the
   following guidelines. Feel free to implement it in other ways that do 
   not involve using the built-in `HashingTF()` or exporting the `text` out of the RDD.
   The goal here is to practice using functional programming to perform
   more complex tasks.
   
   **Guidelines:**
   - **Obtain a count of the vocabulary (unique words) in the whole corpus**
     - Use [Accumulators](https://github.com/apache/spark/blob/master/python/pyspark/accumulators.py)
     - By default `accumulators` have a initial value of `0`. 
     - Import `AccumulatorParam` and make a new class to make the initial value be a `Counter()`
     - Write and map a function to the RDD to update the accumulator. Remember to do a `count` for
       the map to take effect (lazy evaluation)
   - **Make a list of vocab with the top 10,000 count**
     - Use `most_common` on the `Counter`
   - **Obtain TF for each document**
     - Map a function that counts the occurrences of the vocabs given a list of words in a document
     - Normalize the count by the number of words in the document
   
6. Check you TF implementation by using the built-in `HashingTF()` function (see below). The results
   you get from `HashingTF()` should be similar to your implementation. Read the source code for
   `HashingTF()` [(here)](https://github.com/apache/spark/blob/master/python/pyspark/mllib/feature.py)
   and explain why `HashingTF()` might give you a different TF (especially if the specified number of 
   features is low). 
     
   ```python
   ### Using HashingTF() ###
   htf = HashingTF(10000)
   word_vecs_rdd = words_rdd.mapValues(htf.transform)
   word_vec_rdd.count()
   ```
             
7. Spark uses a `LabeledPoint(target, feature)` object to store the target (numeric)
   and features (numeric vector) for all machine learning alogrithm. 
   Map `LabeledPoint(target, feature)` to the RDD and keep the returned RDD in 
   cache using `persist()`. The `feature` is the TF vector.
   
7. Use `randomSplit()` (RDD built-in method) to do a train test split of `70:30`.

8. Train the `NaiveBayes` model on the train data set. See the 
   [docs](http://spark.apache.org/docs/1.2.0/api/python/pyspark.mllib.html#module-pyspark.mllib.classification)
   here.
   
9. Map the `predict()` function of the `NaiveBayes` model onto the test set features to get
   predictions. Caculate the accuracy of the predictions.

10. Examine the predictions that are incorrect. Use the dictionary you have created in `3.` to get
    the `label_name` of those predictions. Examine the content of the incorrect predictions and
    try to reason why the content is incorrectly predicted.

11. Save your model as a pickle file. This will allow you to use the model make prediction about the 
    label of new articles.

12. To make prediction of new data, write a standalone python script that would take the file name
    of the file containing the new articles as a command line argument (use `argparse`).
    
    The script should do the following:
    - Read and preprocess the new data (`data/news_test.pkl`)
    - Preprocessing must be the same as you have done to build the model above
    - Put the new data in an RDD
    - Find the 20 most common words in each article
    - Unpickle your model and make predictions for the label of the article
    - Write out a file with the predicted labels and the most common words 