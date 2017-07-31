---
title: GSOC Sugar Labs - Week 13
layout: post
categories: blog/
excerpt: 'GSOC Suar Labs - Week 13'
tags: [programming,sugar labs,gsoc]
comments: true
description: 'GSOC Suar Labs - Week 13 '

---
This is the thirteenth post in the series of my weekly GSOC Sugar Labs, where I summarize my week of working with [Sugar Labs](https://www.sugarlabs.org) under GSOC.

## Weekly Journal 
Good news, I passed the second evaluation :smile:
This week Samuel and I focused on adding more features to aslo.
We added
* Search (Activities can be searched and supports all languages) 
* Pagination (Index, Search, every user facing template is paginated)
* Testing Core Activities with Fedora 26 (Almost all core activities work with Fedora 26)

### Search
Search is the most basic feature that users will need, given that there are many activities to search for. 
User should be able to navigate to activities they need as fast possible. Currently only name based search is available, search by `bundle_id` will be added later. Originally search was done against `en` or `en_US` locale, but now all locales are supported. If a user is using `hi` (Hindi) version of aslo, names will be matched against hindi name (if any).

{% highlight python %}
def search_by_activity_name(activity_name, lang_code):
    lang_filter_query = me.Q(**{'name__' + lang_code + '__exists': True})
    name_match_query = me.Q(
        **{'name__' + lang_code + '__icontains': activity_name})
    return Activity.get_all().filter(lang_filter_query and name_match_query)
{% endhighlight %}

Since language name can be dynamic as well and Mongoengine follows Djanog's way of handling query. I used `kwargs` to build up dynamic queries and Mongoengines [Query Tree](http://motorengine.readthedocs.io/en/latest/getting-and-querying.html#querying-with-q) `Q` for boolean logic. Search comparison looks for substring inside the name and is not case_sensitive (`icontains`). To ensure language specific result, we look if that activity is available in the stipulated language (`__exists`) . 

Here are some results 

##### Tamil Search
<img src="/images/gsoc-week-13/tamil_search.png" alt="Tamil Search">

<hr>

##### Hindi Search
<img src="/images/gsoc-week-13/hindi_search_t.png" alt="Hindi Search">




# Blog writing in progress


## Goals for Next Week
My plans for next week are 
1. Test core activities ([https://github.com/sugarlabs/sugar-build/blob/master/build/modules.json](https://github.com/sugarlabs/sugar-build/blob/master/build/modules.json) activities start at line 90 )
2. Add Search (Searching by bundle_id or activity name)
3. Improve UI (Add Github Repo Information, make it more beautiful)
4. i18n (Make language specific routes like `/en` or `es/` and translate the aslo UI :smile:)
5. Test the aslo build container with Fedora 26
