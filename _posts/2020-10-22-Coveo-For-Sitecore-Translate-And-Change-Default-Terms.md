---
title: Coveo for Sitecore Translate and Change Default Terms
categories:
- blog
excerpt: Changing Coveo for Sitecore Default Terms Directly in the CMS
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

Coveo for Sitecore Hive Framework is a great framework which is "the entire Coveo JavaScript Search Framework as Sitecore MVC presentation items". Simply, Coveo for Sitecore comes with pre-built Sitecore components and appropriate datasource item templates which you can start using right away without any coding to build your search page. The problem with many of these pre-built Sitecore components and datasource item templates is that not everything is customizable.

## Example

As an example exercise, let's override the terms which are highlighted below.
![](/assets/images/blog/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms-01.png)

By the end of this exercise, you should have these terms replaced with something different like this:
![](/assets/images/blog/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms-02.png)

## The Solution

### JavaScript Override
First, we need to find out if Coveo has a recommended way of changing or overriding the default strings. From their Coveo JavaScript Search Framework documentation, we can find the ["Change the Language of Your Search Interface, Adding or Overriding Strings"](https://docs.coveo.com/en/421/javascript-search-framework/change-the-language-of-your-search-interface#adding-or-overriding-strings) section.
It shows a JavaScript code example showing how to override an existing string.

```js
String.toLocaleString({ 
  "en": {
    "Forward": "Full Speed Ahead!", // Overrides an existing string.
    "MyCustomStringId": "Foo"       // Defines a new string.
  }
});
```
It also notes that
> If you write code to add or override default string values, you should always make sure this code executes after the CoveoJsSearch.js and localization file references in your HTML page.

### Key-Value Pair

All the keys and default terms are available [here](https://github.com/coveo/search-ui/blob/afe7a570735c6f1d11438fd22b315b512b730271/strings/strings.json).

The following shows an example of all the default key-value pairs that come with the installation of Coveo for Sitecore (stored under Website\Coveo\Hive\js\cultures\en.js)
![](/assets/images/blog/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms-03.png)

### Connecting JavaScript Override Script with the Sitecore CMS

We can utilize this script snippet and ASP.NET Razor syntax's ability to dynamically build JavaScript with the data coming from the CMS.

First, I created the Dictionary Resource Rendering Model to keep track of the folder which holds all the dictionary items, the dictionary items themselves stored as Dictionary<string, string> key-value pairs, and the current language because the script snippet requires a two-character language code.

I made a separate datasource template and item to keep the dictionary folder path, where that folder holds all the key-value pairs. I wanted the folder path to be editable but you might come up with different implementation ideas. The important part here is somehow storing the key-value pairs of only the Coveo related dictionary items. If you store all the dictionary items, including the non-Coveo related items, the script will be massive and filled with non-existing key-value pairs.

In my implementation, I used a custom dictionary item template (instead of the out of the box Sitecore one) and stored them under a custom settings folder under the Home item. This allows content editors with no access to the System item to edit these values. Instead of having two fields for key and value, I only created the value field and used the name of the item for the key. You can choose to implement it differently.

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using Sitecore.Data.Fields;
using Sitecore.Data.Items;
using Sitecore.Mvc.Presentation;
using Sitecore.Web.UI.WebControls;

namespace Sitecore.Feature.CoveoSearch.Models.Dictionary
{
    public class DictionaryResourceRenderingModel : IRenderingModel
    {
        public Item Item { get; set; }
        public Item DictionaryFolder { get; set; }
        public Dictionary<string, string> DictionaryItems { get; set; }
        public string CurrentLanguage { get; set; }

        public void Initialize(Mvc.Presentation.Rendering rendering)
        {
            DictionaryItems = new Dictionary<string, string>();
            Item = rendering.Item;
            if (Item != null)
            {
                CurrentLanguage = Item.Language.CultureInfo.TwoLetterISOLanguageName;
                ReferenceField referenceField = Item.Fields["Dictionary Folder"];
                if (referenceField != null && referenceField.TargetItem != null)
                {
                    DictionaryFolder = referenceField.TargetItem;
                }
                if (DictionaryFolder != null)
                {
                    foreach (Item dictionaryItem in DictionaryFolder.Children)
                    {
                        var key = dictionaryItem.Name;
                        var phrase = dictionaryItem.Fields["Phrase"]?.Value;
                        if (!string.IsNullOrWhiteSpace(key) && !string.IsNullOrWhiteSpace(phrase))
                        {
                            DictionaryItems.Add(key, phrase);
                        }
                    }
                }
            }
        }
    }
}
```

Once the rednering model was created, I created the Sitecore rendering which dynamically renders the script with key-value override.

```c#
@model Sitecore.Feature.CoveoSearch.Models.Dictionary.DictionaryResourceRenderingModel
@if (Model.DictionaryItems.Count > 0)
{
    <script>
        String.toLocaleString({
            "@Model.CurrentLanguage": {
                @foreach(var item in Model.DictionaryItems)
                {
                    <text>
                        "@item.Key" : "@Html.Raw(item.Value)",
                    </text>
                }
            }
        });
    </script>
}
```

Then I added the rendering and model to Sitecore and associated them.

![](/assets/images/blog/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms-04.png)
![](/assets/images/blog/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms-05.png)

Finally, I added the newly created rendering to the bottom of my MainLayout.cshtml file.

```c#
@Html.Sitecore().Rendering("/sitecore/layout/Renderings/Feature/Coveo Dictionary/Dictionary Resource", new { DataSource = "/sitecore/content/Home/Settings/Coveo Search Dictionary Config" })
```
I then created a dictionary folder and populated it with custom key-value pairs.
Note that these dictionary strings may contain singular and/or plural tags <sn></sn> <pl></pl>
![](/assets/images/blog/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms-08.png)

Once the folders are properly selected, go to the datasource of the rendering and set the dictionary folder to the one corresponding to Coveo. Again, your implementation on getting these dictionary items could be different.
![](/assets/images/blog/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms-09.png)

## Result

Finally, the terms are edited. Note the script section at the bottom of the DOM.
![](/assets/images/blog/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms/2020-10-22-Coveo-For-Sitecore-Translate-And-Change-Default-Terms-10.png)
