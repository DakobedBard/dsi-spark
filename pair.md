##1. Machine Learning in Spark

Machine learning at scale becomes easy with Spark. Here we use
Naive Bayes on Term-Frequencies to predict the class of an article
based on its content.

1. Import these libraries

   ```python
   import string
   import json 
   import pickle as pkl
   from nltk.tokenize import word_tokenize
   from nltk.corpus import stopwords
   from nltk.stem.porter import PorterStemmer
   import pyspark as ps
   from pyspark.mllib.feature import HashingTF
   from pyspark.mllib.regression import LabeledPoint
   from pyspark.mllib.classification import NaiveBayes
   ```

1. Make a `SparkContext()` and load in the text file. You will need
   to parse the lines into an appropriate data structure (Refer
   to the morning sprint if needed).

   ```python
   data_raw = sc.textFile('s3n://newsgroup/news.txt')
   ```
   
2. Make an RDD with `(key, value)` pairs where the key is the `label`
   and the value is the `text`. 

3. Write a `tokenize()` function that would take the article content
   as a string and return the words in a list. 
   
   Your `tokenize()` function should achieve:
   - Lower casing the text
   - Removing puntuations
   - Removing stop words
   - Stemming 
   - These will help:
     ```python
     PUNCTUATION = set(string.punctuation)
     STOPWORDS = set(stopwords.words('english'))
     STEMMER = PorterStemmer()
     ```
   
   Map the function to the value of the RDD, you should not need to access the 
   key here. 

4. Instantiate the class `HashingTF(n)`, where n is the number of vocabulary
   (features) you will build your model on (`n=50000` is a resonable number).
   Use the class method `transform()` to convert the tokens to a vector of
   term-frequency counts of the vocabulary (unique tokens).
   
   **Optional:**
   
   Below is an [abridged](https://github.com/apache/spark/blob/master/mllib/src/main/scala/org/apache/spark/mllib/feature/HashingTF.scala)
   version of the Scala code for `HashingTF()`. Re-implement the `transform()`
   function in PySpark. 
   
   - Input: `pyspark.resultiterable.ResultIterable` (Lists in PySpark), `feature_num` (int)
   - Output: Python list of word counts with length of `feature_num`
   - Use `collections.Counter()` to avoid the `forEach()` loop
   - Test your function with 1 `ResultIterable`
   - Compare your results with the `transform()` method
   
   <br>
   
   ```scala
   class HashingTF(val numFeatures: Int) extends Serializable {
     def this() = this(1 << 20)
   
     /**
      * Returns the index of the input term.
      */
     def indexOf(term: Any): Int = Utils.nonNegativeMod(term.##, numFeatures)
   
     /**
      * Transforms the input document into a sparse term frequency vector.
      */
     def transform(document: Iterable[_]): Vector = {
       val termFrequencies = mutable.HashMap.empty[Int, Double]
       document.foreach { term =>
         val i = indexOf(term)
         termFrequencies.put(i, termFrequencies.getOrElse(i, 0.0) + 1.0)
       }
       Vectors.sparse(numFeatures, termFrequencies.toSeq)
    }
    ```
5. 