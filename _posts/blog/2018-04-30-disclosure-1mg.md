---
title: Responsible Disclosure - 1Mg
categories: blog
excerpt: 'Finding API Keys in the open plus Unauthenticated API consumption - 1Mg'
tags: [programming,netsec]
comments: true
description: 'Finding API Keys in the open plus Unauthenticated API consumption - 1Mg'

---


I recently started discovering and learning about the world of Network Security and Bug Bounty, and it's absolutely addicting, like playing a game. Wandering around to find treasures, getting defeated, accomplishing levels and sometimes getting rewarded. 

## tl;dr 
Here is a tl;dr if you don't have the time to go through whole post.
While doing subdomain enumeration I found [dmg.1mg.com](https://dmg.1mg.com/html/login.html) and found a Google Search API key and an unauthenticated API endpoint.

## How I found the issue

While doing subdomain enumeration for [https://www.1mg.com/](https://www.1mg.com/), I came across their in-house Drug Management System  [dmg.1mg.com](https://dmg.1mg.com). By default, page redirects to `login.html`, let's try signing up [https://dmg.1mg.com/html/signup.html](https://dmg.1mg.com/html/signup.html), nope no luck, there is no signup page. 


Then I looked at Networks being made, and noticed a GET reuest for `config.json`
<img src="/images/disclosure-1mg/network-tab.png">
and it contained API keys and endpoints.
Here's what `config.json` contained

<img src="/images/disclosure-1mg/config-json.png">

``` json

{  "SEARCH_ENGINE_KEY": "014227405161808294265:4qaxa_56cxk" ,    "TICKET_LIMIT": "20" ,    "DMG_API": "https://api.1mg.com/dmg" ,    "GOOGLE_KEY": "AIzaSyAREDxwRpknZvq-mKQ_vadaMnNDP3_rdbk", "LABS_TEST_API": "https://api.1mglabs.com/admin/test"   }

```

and I was like.
<a href="https://imgflip.com/i/1pg3b2"><img src="https://i.imgflip.com/1pg3b2.jpg" title="made at imgflip.com"/></a>


There was one problem though, I don't know where these keys are used. Without having a possible attack secnario there is no point of reporting it. 

Looking at `GOOGLE_KEY` and `SEARCH_ENGINE_KEY`, it was hinting at a Google Service that uses search engine key, and then I found it, it was [Google Custom Search API](https://developers.google.com/custom-search/json-api/v1/overview).


Looking at documentation, I was able to figure out how to consume the api. 
Let's make a GET request to `https://www.googleapis.com/customsearch/v1?q={QUERY}&key={API_KEY}` but  complains about missing paramter cx (The custom search engine ID to scope the search query (string)), so cx is the `SEARCH_ENGINE_KEY`. 

Let's craft the new GET request as follows  `https://www.googleapis.com/customsearch/v1?q={QUERY}&key={API_KEY}&cx={CUSTOM_KEY}`

So I made a GET to the following endpoint 
`https://www.googleapis.com/customsearch/v1?q=jatin dhankhar&key=AIzaSyAREDxwRpknZvq-mKQ_vadaMnNDP3_rdbk&cx=014227405161808294265:4qaxa_56cx/k`

and I got following in the response. 

<img src="/images/disclosure-1mg/final-query.png">

I tried looking for 1mg's bug bounty/security page but there was none, contacted them via `security@1mg.com`, turns out email didn't exists.

So I sent them a DM on Twitter and they said to send the report to `care@1mg.com` and they will forward to tech team. 1Mg's customer care was very prompt and supportive :+1:. 

Since Google Custom Search has a paid varaint has a paid variant, this was a financial risk. 
5$ for 1000 queries, [which can be done under a minute or two, on a multi threaded system]

1mg fixed the issue by removing and revoking the `GOOGLE_KEY` and `SEARCH_ENGINE_KEY`.

Next I turned my attention to the remaining part of the `config.json`. I opened the `LABS_TEST_API` url  `https://api.1mglabs.com/admin/test` and it said `{"error": "Required params search_text not found"}`. I thought, "Hmm, Interesting, let's give the endpoint what it wants, a search_text parameter". 
So I added the `search_text` parameter with `para` as query (to look for paracetamol :pill:). 
Final url was `https://api.1mglabs.com/admin/test?search_text=para` and response was this.

<img src="/images/disclosure-1mg/dmg-response.png">

Again my reaction was this 

<a href="https://imgflip.com/i/1pg3b2"><img src="https://i.imgflip.com/1pg3b2.jpg" title="made at imgflip.com"/></a>.

I contacted them again and reported the issue and they fixed the issue by adding authentication on the top of the api.


## Bounty
Got a thanks from 1Mg :smile:.
 No rewards, since they don't have bug bounty program, so no **₹**. 

## Timeline
* 27 April, 2018 - Reported API key issue to 1 Mg Customer Care
* 28 April, 2018 - 1 Mg Fixed the issue
* 28 April, 2018 - Reported open API endpoint issue to 1 Mg
* 30 April, 2018 - 1 Mg Fixed the issue
* 30 April, 2018 - Shared the disclosure draft for approval
* 8  May, 2018   - Draft approved and blog published.

## Thanks
Thanks to 1Mg for fixing the bugs and permission to disclose the issues.
1Mg customer care was prompt and supportive, :+1:


