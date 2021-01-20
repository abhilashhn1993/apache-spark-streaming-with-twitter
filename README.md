# Exploring Apache Spark functionalities
This repository includes several works aligned with the aim of exploring the Apache Spark functionalities in the Big Data setting.


## TABLE OF CONTENTS
* [Objective](#objective)
* [Data](#data)
* [Technologies](#technologies)
* [Algorithms](#Algorithm)
* [Implementation](#implementation)
* [References](#references)

## OBJECTIVE
As part of these projects, I explore various concepts underlying Apache Spark such as: 

- RDD creation, manipulation (using transformation and actions)
- creating Spark context & SQL context in Spark
- Performing streaming using Spark

## DATA
The Data for each of the projects is sourced from various sources like Twitter, and a few open source reservoirs. Due to the large size of the data, the files are restrained from uploading to the Git server. 

## TECHNOLOGIES
Python, Apache Spark, PySpark, SQL, tweepy, Streaming, Socket, json

## IMPLEMENTATION
### 1. Data Exploration using Spark and SQL
In this task, Spark transformation and actions are explored on a twitter user data to find the most active users on twitter and find the user with most retweet count

Load the 'Amazon responded' customer data consisting of more than 400 thousand tweets made by the consumers. The data includes the following fields tweet_created_at, user_screen_name, user_id_str

- The above data is explored using Spark actions to find the daily active users by implementing SQL queries in the Spark SQLContext as shown below.
``` python
from pyspark import SparkContext
import pyspark.sql.functions
from pyspark.sql import SQLContext
import pandas as pd
from pyspark.sql import SparkSession
```

```python
sc = SparkContext.getOrCreate()
sqlCon = SQLContext(sc)
```

```python
#Query
result = spark.sql("SELECT value from table2 where value in (SELECT user_id_str FROM table1)")
result = result.withColumn("Whether_active", lit("Yes"))

#Active users
active_users = result.count() #active users in those chosen for the experiment
```
The code file can be found [here](https://github.com/abhilashhn1993/tweet-streaming-with-spark/blob/main/Code/Match-active-users-twitter.ipynb)

### 2. Spark Streaming - Twitter
Building a simple application that reads online streams from Twitter (for a given twitter handle), then processes the tweets using Apache Spark Streaming context created in order to achieve the following tasks
  - listing hashtags
  - highlight the trending hashtag
  - Conduct a simple sentiment analysis using [Bing Liu Opinion Lexicon](https://www.cs.uic.edu/~liub/FBS/sentiment-analysis.html) 
 
This involves a two-step approach:
##### 1. Create a Tweet Listener:  
A connection socket is created from which Tweets are listened (for a given twitter handle/hashtag condition) using a custom Tweet listener created on localhost. The below code block shows the custom function to read data from json (sent by twitter)
  
  ```python
  class TweetsListener(StreamListener):
    
    def __init__(self, csocket):
        self.client_socket = csocket
        self.counter = 0
        self.limit = 1000
        
    def on_status(self, status):
        #print(status.text)
        self.counter+=1
        if self.counter < self.limit:
            return True
        else:
            return False
             
    def on_data(self, data):
        try:
            msg = json.loads(data)
            print(msg['text'].encode('utf-8'))
            self.client_socket.send(msg['text'].encode('utf-8'))
            return True
        except BaseException as e:
            print("Error in on_data: %s" % str(e))
        return True
    
    def on_error(self, status):
        print(status)
        return True
  ```
The full code for the Twitter Listener can be found [here](https://github.com/abhilashhn1993/tweet-streaming-with-spark/blob/main/Code/TweetListener.ipynb)
  
#### 2. Spark Streaming:
The Tweets listened are collected from a Spark context and Streaming context. 
```python
#create spark context and the streaming context
sc = SparkContext("local[2]")
ssc = StreamingContext(sc,10)
IP = "localhost"
port=5555

#Logging control
sc.setLogLevel("ERROR")

lines = ssc.socketTextStream(IP, port)
  ```
#### 3. Calculate Sentiment Score:
The tweets streamed are passed for computation of sentiment scores using Bing-Liu Opinion Lexicon as a reference. Refer the below code
```python
def calculate_sentiment_scores(text):
    tokens = text.split(" ")
        
    neg_score = 0
    pos_score = 0
    
    for word in dict_neg:
        if (word in tokens):
            neg_score = neg_score + 1
    
    for word in dict_pos:
        if (word in tokens):
            pos_score= pos_score + 1
    
    sentiment_score = pos_score - neg_score
    return str(sentiment_score)
```
The sentiment scores calculated for each tweet is then transformed into a Spark RDD and the streaming context is terminated. 

The code file can be found [here](https://github.com/abhilashhn1993/tweet-streaming-with-spark/blob/main/Code/SparkStreamer.ipynb)

## REFERENCES
- https://www.toptal.com/apache/apache-spark-streaming-twitter
- http://davidiscoding.com/real-time-twitter-analysis-3-tweet-analysis-on-spark
- https://medium.com/@distillerytech/bringing-big-tools-to-big-data-spark-and-spark-streaming-ed93f5b478d7


