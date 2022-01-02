---
title: Analyising Stock Market Trends by Crawling Reddit
categories:
- blog
excerpt: Crawling the r/wallstreetbets Subreddit using PRAW Python Reddit API Wrapper and storing keywords in MongoDB. Retrieving aggregate data from MongoDB and graphing keyword trends within specified time range using Pyplot.
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
  overlay_image: /assets/images/blog/2021-04-20-analyising-stock-market-trends-by-crawling-reddit/2021-04-20-analyising-stock-market-trends-by-crawling-reddit-hero.jpg
  overlay_filter: "0.7"
  show_overlay_title: true
---

## Inspiration

I started working on this project back in Janurary, when the infamous subreddit with the name r/wallstreetbets were talking about a single stock that had incrased in value by 57% in one day. This stock was GameStop, or call more often as its stock symbol, GME. Although not an active member, I've been browsing this community well before the GME boom of 2021, just to find some fun side of stock trading and seeing some irresponsible traders showing off their options. Sometimes with gains of 500% or greater in a few days, or losing it all on their margin accounts. This community of foul mouthed and borderline gamblers were discussing how the community could buy GME since the stock was shorted beyond what is usually possible [Reuters - Explainer: How were more than 10% of GameStop's shares shorted? Short squeeze explained](https://www.reuters.com/article/us-retail-trading-shortselling-explainer-idUSKBN2AI2DD). Some were speculating that a short squeeze that happened with VW in October of 2008 could happen again with GME.

I saw the entire situation esclating in front of my eyes. The subreddit's follower count was growing exponentially and mainstream news stations were now covering Reddit. It was clear that r/wallstreetbets was having the most significant impact on the stock price of GME. I wanted to investigate this subreddit further. I came up with an idea to crawl this subreddit and analyise what people were talking about using keywords. I would go through every post and every comment to see which words were being used the most. By plotting the trending keywords, I believed I could be one of the first to get in on the action next time.

## Design

My design for this MVP was extremely simple. Read all the recent posts' comments, find out all the popular keywords, and store that information in a database to be queried later.

### Crawling Reddit

Interacting with the Reddit API isn't anything new. Infact, there is an entire subreddit dedecated to building apps using the Reddit API called r/redditdev. The logic of the code wasn't too difficult. I stored every word used in a dictionary where the word is the key and the value is the number of times it was used in the post. Then I trimmed all the words that weren't used more than x number of times (configurable), and I pushed the data to my database.

@@@@ Code snippet here @@@@

### Database

I used MongoDB for this project as I knew my data would not be structured. The words used in each post will be different from the words used in other posts. Having a JSON like documents to query though was ideal for this project.

@@@@ JSON structure here @@@@

### Querying the Data

I needed an easy way to query a keyword with parameters like start DateTime, end DateTime, and time interval.

@@@@ Example parameters here @@@@

## Result

@@@@ Screenshots and results @@@@
