---
title: Coveo for Sitecore Clear All Facets on New Query
categories:
- blog
excerpt: New search term means new fliters too!
tags:
- Sitecore
- Coveo
- Coveo For Sitecore
- Search
- Versioning
- Translation
date: 2020-11-02T04:00:00.000+00:00
header:
  overlay_image: /assets/images/blog/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms-Hero.png
  overlay_filter: "0.7"
  show_overlay_title: true
---

## The Problem

Coveo for Sitecore Hive Framework lets developers utilize the crucial functions of Coveo search. With the installation of Coveo for Sitecore, you get many components and their corresponding datasource items containing all sorts of configuration settings. Coveo for Sitecore Search Interface component is one of them. It has many settings which controls search behaviours. However, there are some functionalities which might seem trivial but are missing from the settings. In my opinion, clearing the facet when a new search query is performed is one of them. As of now, Coveo for Sitecore does not provide a setting where facets reset after a new query.
Let's think of a scenario where we would want a new search query to reset the facets (commonly called filters). For example, let's say we are on a food ordering app searching for something to eat for dinner. I have tomato juice in mind so I search for "tomato" then select "drinks" in the category filter. I found what I wanted so I search for my main meal. I search for "pasta" in the search section but I get no results back because there's nothing that has the word "pasta" in the "drinks" category. I uncheck the "drinks" category and I finally see what I wanted.
I think there are many cases which reseting all the facets is important, like the case I just described.
Now, let's see this behaviour and see how we can resolve it.

## Example

### Before
![](/assets/images/blog/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms/2020-10-05-Coveo-For-Sitecore-Clear-All-Facets-On-New-Query-01.gif)

### After
![](/assets/images/blog/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms/2020-10-05-Coveo-For-Sitecore-Clear-All-Facets-On-New-Query-02.gif)
## The Solution
