##Part 1: Machine Learning in Spark

Here we are going to run some Machine Learning Algorithms using MLlib, the Machine
Learning library for Spark. We are going to build a Naive Bayes model to predict
the category of a newsgroup article based on its content.

<br>

1. Start a local cluster just like you did in the morning assignment. Working from
   the IPython notebook associated with that cluster, import these libraries:

   ```python
   import string
   import json
   import pickle as pkl
   from pyspark.mllib.feature import HashingTF
   from pyspark.mllib.regression import LabeledPoint
   from pyspark.mllib.classification import NaiveBayes
   from collections import Counter
   ```

2. Load the text file into an RDD, and map the lines to dictionaries using
   `json.loads()`. `take()` the first 2 lines to confirm your results.

   ```python
   data_raw = sc.textFile('s3n://[YOUR_AWS_ACCESS_KEY_ID]:[YOUR_AWS_SECRET_ACCESS_KEY]@sparkdatasets/news.txt')
   ```

3. Check how many partitions are in your RDD by using `.getNumPartitions()`. If
   it is too low (i.e. 2), you might want to repartition your RDD with
   [`.repartition()`](https://spark.apache.org/docs/1.1.1/api/python/pyspark.rdd.RDD-class.html#repartition) to a higher number of partitions. While partitioning
   beyond the number of cores you have does not in theory increase parallelism,
   it allows you to track your tasks within a job more easily (since this leads to
   smaller tasks).

4. There are 3 fields in the data: `label`, `label_name`, and `text`.

   - Make an RDD of unique `(label, label_name)` pairs. This creates a reference
     of the numeric label to the name of the label. We will need it later.

   - Make another RDD with `(label, text)`. We will make a bag-of-words
     and compute term-frequency out of the `text`. The label will be the
     target we will later train our Naive Bayes on, and the term-frequency
     will be the feature.

5. Tokenize your text by mapping the following functions:

   - Lower casing the text
   - Removing punctuations
   - Removing stop words
   - Splitting on white spaces
   - Stemming the tokenized words

   Feel free to use `nltk`. Also feel free to wrap all of the functions in one
   big function and then map that over your RDD. If you get stuck or bogged down
   with errors, try building up and mapping your function across the RDD one piece
   at a time, checking with `.take(1)` that you get your intended results after
   each piece.

6. You should now have a `(key, value)` pair RDD where the `key` is the numeric
   value that represents the category (i.e. label) of newsgroup of the article and the `value`
   is a list of the stemmed words of the article.

   Create a new RDD where the key remains the category and the value is the
   term-frequency (TF) vector for each article. This is going to require some
   thinking, and comes with some caveats. You may not use the built-in
   `HashingTF()` (we'll use this in a second), nor use `collect()` on the
   whole corpus. You are only allowed to do a `collect()` of an RDD that
   contains the vocab of the corpus. This exercise will be good practice for
   when you want to perform other very complicated manipulations on an RDD for
   which there are not built-ins like the `HashingTf()`.

   Here are some functions you might want to consider (not necessarily in order)
   to implement TF.

   - `.values()`
   - `.flatMap()`
   - `.distinct()`
   - `.mapValues()`
   - `Counter()` (Python class)

7. Now let's use `HashingTF()` to get the word vector in order to confirm
   that your implementation is correct. Your first row should have a sum of 186
   words. You cannot compare the actual word vec since `HashingTF()` would have
   mixed up the ordering of the vocab.

      ```python
   ### Using HashingTF() with 10000 features ###
   htf = HashingTF(10000)
   word_vecs_rdd = words_rdd.mapValues(htf.transform)
   word_vecs_rdd.count()
   ```

   One advantage with your implementation is that you have access to the actual
   vocab itself, whereas `HashingTF()` has obscured the words by hashing them.

8. Spark uses a `LabeledPoint(target, feature)` object to store the target (numeric)
   and features (numeric vector) for all machine learning algorithms. So that we
   can fit a NaiveBayes, make a new RDD by mapping `LabeledPoint(target, feature)`
   to the RDD and keep the new RDD in cache using `persist()`. Remember to `setName()`
   before you persist. The `feature` is the TF vector.

9. Use `randomSplit()` (RDD built-in method) to do a train test split of `70:30`.

10. Train the `NaiveBayes` model on the train data set. See the [docs](http://spark.apache.org/docs/1.2.0/api/python/pyspark.mllib.html#module-pyspark.mllib.classification)
    here. The  `NaiveBayes` model is a Python class, and once it is trained, it
    can be used like a Python class.

11. Map the `predict()` method of the `NaiveBayes` model onto the test set
    features to get predictions. Calculate the accuracy of the predictions.
    Your accuracy should be above 80%.

    If this is taking too long. Reference the timeline of my runtime shown below.

    ![image](images/log.png)

12. Examine the predictions that are incorrect. Use the dictionary you have created
    in `4.` to get the `label_name` of those predictions. Examine the content of
    the incorrect predictions and try to reason why the content is incorrectly
    predicted.

If you'd like to read more about textual feature extraction and implementing
tf-idf using the PySpark built-ins, check out [this resource](https://spark.apache.org/docs/1.2.0/mllib-feature-extraction.html).

<br>

##Extra Credit: Word2Vec

Spark also has a [Word2Vec implementation](https://spark.apache.org/docs/1.2.0/mllib-feature-extraction.html). Word2Vec is
the name given to a class of neural network models that, given an unlabelled
training corpus, produces a vector for each word in the corpus that encodes
its semantic information. Word2Vec is a technique to get vector representations
of words through unsupervised training (i.e regardless of the category / class
of the article). Word2Vec has 2 implementations:

* Using the [skip-gram model](http://alexminnaar.com/word2vec-tutorial-part-i-the-skip-gram-model.html), where given 1 word, we try to
predict the likelihood of surrounding words.
* Using a continuous bag of words, where the likelihood of the current word is
being predicted based on neighboring words.

Both of these models are simple neural networks with one hidden layer. The word
vectors are learned via backpropagation and stochastic gradient descent

<br>

1. To train a Word2Vec model, we need training data. Load in the data as follows:

   ```python
   data = sc.textFile('s3n://[YOUR_AWS_ACCESS_KEY_ID]:[YOUR_AWS_SECRET_ACCESS_KEY]@sparkdatasets/text8_lines')
   ```

2. Import and instantiate a `Word2Vec()` class.

   ```python
   from pyspark.mllib.feature import Word2Vec
   word2vec = Word2Vec()
   ```

3. Fit the `Word2Vec()` class with an RDD where each item is a list of words
   from each article. The model will take a while to train (~10 - 20 mins).
   However, once the model is trained, it does not have to be re-trained.
   It can be used as a normal Python class.

4. One of common uses of Word2Vec is to find words similar in context to a word
   in question. Call the `findSynonyms()` function of the model and provide the
   word in question as the first argument and the number of most similar words
   to extract as the second argument.

5. The result is a tuple of `(words, cos similarity)`. Print out the words closest
   to `general`. Try out other words you like.

6. The `Word2Vec` class also has transform method that takes a word and transforms
   it to the word vector which is the output of the hidden layer as discussed. This
   allows us to do other things such as clustering words that potentially have
   similar context.

7. Check out [Gensim's](http://radimrehurek.com/2014/02/word2vec-tutorial/)
   Word2Vec implementation as well if you are interested.
