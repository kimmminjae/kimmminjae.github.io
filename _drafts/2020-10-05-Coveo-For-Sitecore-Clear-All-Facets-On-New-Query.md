---
title: Coveo for Sitecore Clear All Facets on New Query
categories:
- blog
excerpt: New search term means new filters too!
tags:
- Sitecore
- Coveo
- Coveo For Sitecore
- Search
- Facet
- Filter
- Clear Filter
date: 2020-10-05T04:00:00.000+00:00
header:
  overlay_image: /assets/images/blog/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms-Hero.png
  overlay_filter: "0.7"
  show_overlay_title: true
---

## The Problem

Coveo for Sitecore Hive Framework lets developers utilize many crucial functionalities of Coveo Search JavaScript Framework. With the installation of Coveo for Sitecore, you get many components and their corresponding datasource items containing all sorts of configurations which controls the behaviour of the search page. Coveo for Sitecore Search Interface component is one that encompasses all the search components (search bar, search results, facets, sorts, etc.). However, there are some functionalities which might seem trivial but are missing from the configuration item. In my opinion, clearing all the facets when a new search query is performed is one of them. As of now, Coveo for Sitecore does not provide a setting to reset all the facets after a new query.

## Example

Let's think of a scenario where we would want a new search query to reset all the facets (commonly called filters). For example, let's say we are on a food ordering app searching for something to eat for dinner. I have tomato juice in mind so I search for "tomato" then select "drinks" in the category filter. I found what I wanted so I search for my main meal, "pasta", in the search section but I get no results back because there's nothing that has the word "pasta" in the "drinks" category. I uncheck the "drinks" category and I finally see what I wanted.
I think there are many cases which resetting all the facets is useful to the users, like the case I just described.
Now, let's see this behaviour and see the end product as a comparison.

### Before
![](/assets/images/blog/2020-10-05-Coveo-For-Sitecore-Clear-All-Facets-On-New-Query/2020-10-05-Coveo-For-Sitecore-Clear-All-Facets-On-New-Query-01.gif)

### After
![](/assets/images/blog/2020-10-05-Coveo-For-Sitecore-Clear-All-Facets-On-New-Query/2020-10-05-Coveo-For-Sitecore-Clear-All-Facets-On-New-Query-02.gif)

## The Solution

We need to change a couple of things to make our search page clear all the filters upon a new query.
1. Check if the previous search query was different than the new one
2. Clear the facet if it was different then execute a query

We don't want to check after we execute the query since that will cost one additonal query (counted towards the query limit), and will cost performance, time, and have weird animations in between.
Instead, we want to check for all these in-between queries.
That's were ["Coveo Javascript Framework Query Events"](https://docs.coveo.com/en/417/javascript-search-framework/javascript-search-framework-events#query-events) comes in.
We can tap into one of many events and insert our logic during queries.

The perfect place to tap in is during the "buildingQuery" event. There are lots of things going on during "buildingQuery" and applying the selected facets into the query being sent to the Coveo Cloud server is one of them. Also, we can find the new query string during "buildingQuery" so it's a perfect place to perform all our tasks.

Below, I get the Coveo Search Interface ID, then listen to the "buildingQuery" event. Within the event, I check for the current search term and compare it with the previous term. If it's different from the previous term and the previous term was not an empty string, I clear all the facets. Note, the images I used for my example has a "CoveoFacetSlider" component. The code shown below doesn't clear the slider facets. If you wish to clear the slider facet as well, or any other facet type, simply extend the code to include those facet components.

Below, I list two versions. Using pure JavaScript, and using jQuery.
"CoveoCategoryFacet" component uses .resetPath() instead of .reset() to reset its values. .reset() results in infinite search in this case. Looking further into the source code for .reset() of "CoveoCategoryFacet", I found that it executes a query after running .resetPath(). I presume that .executeQuery() gets run recursively due to it being triggered from "buildingQuery" (and building query gets run with every query execution). I prevent this by just running .resetPath().


### Vanilla JavaScript Version
``` js
var previousSearchTerm = "";
document.addEventListener("DOMContentLoaded", function () {
    var root = document.querySelector("#coveoC606340E");
    if (root != undefined) {
        Coveo.$$(root).on('buildingQuery', function (e, args) {
            var currentSearchTerm = Coveo.state(root, 'q')

            if (previousSearchTerm != currentSearchTerm && previousSearchTerm != "") {
                var coveoFacets = Coveo.$$(document).findAll('.CoveoFacet');
                if (coveoFacets) {
                    coveoFacets.forEach(coveoFacet => Coveo.get(coveoFacet).reset());
                }
                var coveoFacetRanges = Coveo.$$(document).findAll('.CoveoFacetRange');
                if (coveoFacetRanges) {
                    coveoFacetRanges.forEach(coveoFacetRange => Coveo.get(coveoFacetRange).reset());
                }
                var coveoCategoryFacets = Coveo.$$(document).findAll('.CoveoCategoryFacet');
                if (coveoCategoryFacets) {
                    coveoCategoryFacets.forEach(coveoCategoryFacet => Coveo.get(coveoCategoryFacet).resetPath());
                }
            }
            previousSearchTerm = currentSearchTerm;
        });
    }
});
```

### jQuery Version
``` js
var previousSearchTerm = "";
document.addEventListener("DOMContentLoaded", function () {
    var root = document.querySelector("#coveoC606340E");
    if (root != undefined) {
        Coveo.$$(root).on('buildingQuery', function (e, args) {
        var currentSearchTerm = Coveo.state(root, 'q')
            if (previousSearchTerm != currentSearchTerm && previousSearchTerm != "") {
                Coveo.$('.CoveoFacet').coveo('reset');
                Coveo.$('.CoveoFacetRange').coveo('reset');
                Coveo.$('.CoveoCategoryFacet').coveo('resetPath');
            }
            previousSearchTerm = currentSearchTerm;
        });
    }
});
```

### Note
Make sure to replace the Coveo Search Interface ID with your Search Interface ID
``` js
var root = document.querySelector("#YourCoveoSearchInterfaceID");
```
![](/assets/images/blog/2020-10-05-Coveo-For-Sitecore-Clear-All-Facets-On-New-Query/2020-10-05-Coveo-For-Sitecore-Clear-All-Facets-On-New-Query-03.png)

## Result

Finally, when you search with a new query string, all the facets will reset!

![](/assets/images/blog/2020-10-05-Coveo-For-Sitecore-Clear-All-Facets-On-New-Query/2020-10-05-Coveo-For-Sitecore-Clear-All-Facets-On-New-Query-02.gif)
