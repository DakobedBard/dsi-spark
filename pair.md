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
   from nltk.tokenize import word_tokenize
   from nltk.corpus import stopwords
   from nltk.stem.porter import PorterStemmer
   from pyspark.mllib.feature import HashingTF
   from pyspark.mllib.regression import LabeledPoint
   from pyspark.mllib.classification import NaiveBayes
   from pyspark.accumulators import AccumulatorParam
   from collectons import Counter
   ```

1. Make a `SparkContext()` and load the text file into an RDD. Map the lines
   to dictionaries. Take the first 2 lines to confirm your results.

   ```python
   data_raw = sc.textFile('s3n://newsgroup/news.txt')
   ```
   
2. There are 3 fields in the data: `label`, `label_name` and `text`.
   Make an RDD of unique `(label, label_name)` key / value pairs.
   Collect the RDD to a python dictionary. That is needed for reference later.
   
   Make another RDD with `(label, text)`. 

3. Write a `tokenize()` function that would take the article content
   as string and return a list of words. 
   
   Your `tokenize()` function should achieve:
   - Lower casing the text
   - Removing puntuations
   - Removing stop words
   - Stemming 
   - Splitting on white spaces
   - Consider using the following:
     ```python
     PUNCTUATION = set(string.punctuation)
     STOPWORDS = set(stopwords.words('english'))
     STEMMER = PorterStemmer()
     ```
     
   Map the function to the `text`.

4. Transform the tokenized text to term-frequency vectors. Consider the
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
     - Write a function that counts the occurrences of the vocabs given a list of words in a document
   - **Check your implementation**
     - Compare to the `HashingTF()` function.
     
       ```python
       ### Using HashingTF() ###
       htf = HashingTF(10000)
       word_vecs_rdd = words_rdd.mapValues(htf.transform)
       word_vec_rdd.count()
       ```
           
5. Spark uses a `LabeledPoint(target, feature)` object to store the target (numeric)
   and features (numeric vector) for all machine learning alogrithm. 
   Map `LabeledPoint(target, feature)` to the RDD and keep the returned RDD in 
   cache using `persist()`. The `feature` is the TF vector.
   
6. Use `randomSplit()` (RDD built-in method) to do a train test split of `70:30`.

