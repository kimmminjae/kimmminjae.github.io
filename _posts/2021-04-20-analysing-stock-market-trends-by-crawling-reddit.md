---
title: Analysing Stock Market Trends by Crawling Reddit
categories:
- blog
excerpt: Crawling the r/wallstreetbets subreddit using PRAW Python Reddit API Wrapper and storing keywords in MongoDB. Retrieving aggregate data from MongoDB and graphing keyword trends within a specified time range using Pyplot.
tags:
- Data
- Crawling
- Python
- MongoDB
- Data Science
- Reddit
- PRAW
date: 2021-04-20T04:00:00.000+00:00
header:
  overlay_image: /assets/images/blog/2021-04-20-analysing-stock-market-trends-by-crawling-reddit/2021-04-20-analysing-stock-market-trends-by-crawling-reddit-hero.jpg
  overlay_filter: "0.8"
  show_overlay_title: true
---

## Inspiration

I started working on this project back in January when the infamous subreddit with the name r/wallstreetbets was talking about a single stock that had increased in value by 57% in one day. This stock was GameStop, or called more often as its stock symbol, GME. Although not an active member, I've been browsing this community well before the GME boom of 2021, just to find some fun side of stock trading and see some irresponsible traders showing off their options. Sometimes with gains of 500% or greater in a few days, or losing it all on their margin accounts. This community of foul-mouthed and borderline gamblers were discussing how the community could buy GME since the stock was shorted beyond what is usually possible [Reuters - Explainer: How were more than 100% of GameStop's shares shorted? Short squeeze explained](https://www.reuters.com/article/us-retail-trading-shortselling-explainer-idUSKBN2AI2DD). Some were speculating that a short squeeze that happened with VW in October of 2008 could happen again with GME.

I saw the entire situation escalating in front of my eyes. The subreddit's follower count was growing exponentially and mainstream news stations were now covering Reddit. It was clear that r/wallstreetbets was having the most significant impact on the stock price of GME. I wanted to investigate this subreddit further. I came up with an idea to crawl this subreddit and analyse what people were talking about using keywords. I would go through every post and every comment to see which words were being used the most. By plotting the trending keywords, I believed I could be one of the first to get in on the action next time.

## Design

My design for this MVP was extremely simple. Periodically read all the recent posts' comments, find out all the popular keywords, and store that information in a database to be queried later.

![System design diagram](/assets/images/blog/2021-04-20-analysing-stock-market-trends-by-crawling-reddit/2021-04-20-analysing-stock-market-trends-by-crawling-reddit-system-diagram.png)

### Crawling Reddit

Interacting with the Reddit API isn't anything new. In fact, there is an entire subreddit dedicated to building apps using the Reddit API called r/redditdev. The logic of the code isn't too complex. I stored every word used in a dictionary where the word is the key and the value is the number of times it was used in the post. Then I trimmed all the words that weren't used more than x number of times (configurable), and I pushed the data to my database. All the common words like "a", "the", "they", "have", etc. are discarded.

First, I connect to my MongoDB collection that will store my word count data.
``` python
client = MongoClient("mongodb+srv://abcxyz@cluster0.0mlon.mongodb.net/submissions_db?retryWrites=true&w=majority")
db = client.get_database()
records = db.submission_records
```

Connecting to the subreddit.
``` python
reddit = praw.Reddit(client_id='abc', client_secret='def', user_agent='ghi', username='jkl', password='mno')
subredditname = "wallstreetbets"
subreddit = reddit.subreddit(subredditname)
```

Retreveing common words from the commonwords.tex file.
``` python
with open("commonwords.txt", "r") as words:
    commonwords = [word.lower().rstrip() for word in words]
```

Take each submission (post) from a subreddit and crawl the comments.
``` python
def get_word_count_object(submissions_list, ignored_commonwords, min_score):
    word_count_object = []
    skip_submission_list = []

    # Iterate over each submission group (top of the week, top of the day, etc.) in the subreddit
    for submissions in submissions_list:
        # Iterate over each submission within each group
        for submission in submissions:
            # Skip if submission has been crawled already
            if not submission.id in skip_submission_list:
                # Add the current submission to the skip submission list
                skip_submission_list.append(submission.id)
                words = []
                word_count = {}
                # skip if the submission has less than minimum submission score
                if submission.score >= min_score:
                    # Load all comments
                    submission.comments.replace_more(limit=0)
                    # Iterate over each comment
                    for top_level_comment in submission.comments:
                        word = ""
                        # Build the word by going over each character
                        for letter in top_level_comment.body:
                            if(letter == ' '):
                                if(word and not word[-1].isalnum()):
                                    word = word[:-1]
                                # Add the word to the words list if it's not in the commonwords.txt file
                                if not word.lower() in ignored_commonwords:
                                    words.append(word)
                                word = ""
                            else:
                                # Adds the lower case version of the letter if the letter is not a special character
                                word += letter_filter(letter)
                    # Create a dictionary with word as the key and count as the value (code simplifiable using defaultdict)
                    for word in words:
                        if word in word_count:
                            word_count[word] += 1
                        else:
                            word_count[word] = 1
                    word_count = { k:v for k,v in word_count.items() if v > 1 }
                    # Append submission result to the word_count_object
                    word_count_object.append({
                        '_id' : submission.id,
                        'created_utc' : int(submission.created_utc),
                        'word_count' : word_count
                    })
            else:
                print('skipped')
    return word_count_object
```

Finally, use the function shown above by getting the top 200 posts of the last 7 days, top 100 posts of the last 24 hours, and top 10 posts of the last 1 hour. Then upload the data to the database.
``` python
while True:
    print('Start scraping...', flush=True)
    word_count_object = get_word_count_object(submissions_list = [subreddit.top(limit=200, time_filter="week"), subreddit.top(limit=100, time_filter="day"), subreddit.top(limit=20, time_filter="hour")], ignored_commonwords = commonwords, min_score = 1000)
    # Upload or update the submission result in the DB.
    for word_count in word_count_object:
        print(word_count['_id'])
        records.replace_one({'_id' : word_count['_id']}, word_count, upsert=True)
```


### Database

I used MongoDB for this project as I knew my data would not be structured. The words used in each post will be different from the words used in other posts. Having JSON like documents to query though was ideal for this project.

The following example is taken from MongoDB and is an actual example. The related post can be checked at [http://reddit.com/g55or2](http://reddit.com/g55or2).

``` javascript
{
  _id: "g55or2",
  created_utc: 1587431901,
  word_count:
  {
    barrel: 9,
    oil: 49,
    supreme: 17,
    .
    .
    .
    consumer: 2,
    selling: 3
  }
}
```

![Screenshot of MongoDB Atlas with list of documents representing subreddit submissions](/assets/images/blog/2021-04-20-analysing-stock-market-trends-by-crawling-reddit/2021-04-20-analysing-stock-market-trends-by-crawling-reddit-mongodb-atlas-data-example.png)

Screenshot of MongoDB Atlas with a list of documents representing subreddit submissions

### Querying the Data

I needed an easy way to query a keyword with parameters like start time, end time, and time interval.

First, I connect to my MongoDB collection containing my word count data.
``` python
client = MongoClient("mongodb+srv://abcxyz@cluster0.0mlon.mongodb.net/submissions_db?retryWrites=true&w=majority")
db = client.get_database()
records = db.submission_records
```

Then, define my function to search for a specific keyword between two time intervals. The start and end times are in [Unix time](https://www.unixtimestamp.com/).

``` python
# Searches the number of times the 'keyword' parameter was used between the given start and end time.
def keyword_searcher(records, keyword, start_time, end_time):
    word_count = '$word_count.' + str(keyword)
    group = records.aggregate([
        {
            '$match' : {
                '$and' : [
                    {'created_utc' : { '$gte' : start_time }},
                    {'created_utc' : { '$lte' : end_time }}
                ]
            }
        },
        {
            '$group': {
                '_id' : None,
                'word_count' : { '$sum' : word_count }
            }
        },
        {
            '$sort' : { '_id' : 1 }
        }
    ])
    for item in group:
        return item['word_count']
```

Finally, we get the desired result by iteratively calling the 'keyword_searcher' function through the start and end time, increasing each by the interval.
``` python
# Searches the number of times the 'keyword' parameter was used between the given start and end time, dividing the total time by the 'interval'.
def keyword_count_time_range(records, keyword, start_time, end_time, interval):
    result = []
    range_count = int(round((end_time - start_time) / interval))
    for i in range(range_count):
        start_time = start_time + (interval * i)
        end_time = start_time + (interval * (i + 1)) -1
        result.append({ 'count' : keyword_searcher(records, keyword, start_time, end_time), 'start_time' : datetime.fromtimestamp(start_time), 'end_time' : datetime.fromtimestamp(end_time)})
    return result
```

With the result, we can utilize the data in our own ways. To visualise the data, we can plot it.
``` python
result = keyword_count_time_range(records, keyword, start_time, end_time, interval)
times = []
counts = []

for item in result:
    times.append(item['start_time'])
    counts.append(item['count'])

plt.style.use('seaborn')
plt.plot_time(times, counts, linestyle='solid')
plt.tight_layout()
plt.show()
```

## Result

![Data plot of the keyword 'gme' for the past 60 days from March 13, 2021 with interval of 24 hours](/assets/images/blog/2021-04-20-analysing-stock-market-trends-by-crawling-reddit/gme-60-days-24-hour-20210313.png)

Data plot of the keyword 'gme' for the past 60 days from March 13, 2021 with interval of 24 hours

![Data plot of the keyword 'gme' for the past 90 days from March 13, 2021 with interval of 24 hours](/assets/images/blog/2021-04-20-analysing-stock-market-trends-by-crawling-reddit/gme-90-days-24-hour-20210313.png)

Data plot of the keyword 'gme' for the past 90 days from March 13, 2021 with interval of 24 hours

![Data plot of the keyword 'gme' for the past 60 days from March 13, 2021 with interval of 12 hours](/assets/images/blog/2021-04-20-analysing-stock-market-trends-by-crawling-reddit/gme-60-days-12-hour-20210313.png)

Data plot of the keyword 'gme' for the past 60 days from March 13, 2021 with interval of 12 hours

![Data plot of the keyword 'amc' for the past 60 days from March 13, 2021 with interval of 24 hours](/assets/images/blog/2021-04-20-analysing-stock-market-trends-by-crawling-reddit/amc-60-days-24-hour-20210313.png)

Data plot of the keyword 'amc' for the past 60 days from March 13, 2021 with interval of 24 hours

## Conclusion

I found this project extremely enjoyable as I was able to work with massive amounts of data and organize them in useful ways. Since the Reddit community was talking about GameStop stock before the "boom", I hoped that capturing the keyword trends would help me capture the next "boom". Although understanding the stock market trends is viable using this tool, nobody should use this as a definitive financial advice tool.

The spikes in the resulting graphs did show correlation the next day. However, the correlation was either a big jump or a big fall in price. Thus, the graphs are more correlated to the next day's trade volume than the correlation of price when solely searching for the stock symbol/name. However, querying keywords such as "buy" or "sell" could indicate the supply and demand of the stock.
