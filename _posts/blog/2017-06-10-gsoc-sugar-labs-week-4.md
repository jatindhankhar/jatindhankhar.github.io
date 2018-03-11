---
title: GSOC Sugar Labs - Week 4
categories: blog/
excerpt: 'GSOC Suar Labs - Week 4 '
tags: [programming,sugar labs,gsoc]
description: 'GSOC Suar Labs - Week 4 '

---

This is the fourth post in the series of my weekly GSOC Sugar Labs, where I summarize my week  of working with [Sugar Labs](https://www.sugarlabs.org) under GSOC.
This post was published little late :sweat_smile: due to the delays on my side, since I had to perform a clean install of my system due to two reasons, I wanted to remove too much cruft that accumulated over time.

## Weekly Journal

This week was the first part of the coding period and I was pretty excited to get on the coding part. After a bit of discussion between Tony, Samuel and Walter, it was decided that we would follow a Event driven model (details of which will be refined and iterated in the coming time), where developer will only push the code to repo and all the handling and building will be done by ASLO. In the end Tony gave Samuel and my idea a chance and Samuel become my mentor (both Tony and Samuel are my mentor but from now on I will be working more closely with Samuel). 
We started working on the ASLO implemetation. A rough sketch of the workflow of ASLO (as of writing )is here.
<img src="/images/gsoc-week-4/flow.png" alt="Brief overview of ASLO workflow" />
(Not a very good looking and accurate flow chart, though :sweat_smile:* )
When a developer release a new version of an activity on an repo of the [Sugar-Activities](github.com/sugar-activities) Organization a payload is sent to the webhook (which happens to at route /webhook of ASLO).
ASLO then intiaties a background task via Celery to clone the repo and a docker container (made from a prebuilt image) is launched to bundle the code into .xo file which can then be stored on centralized location from where users can download it. Storage location for bundles can be on the same server where ASLO is running or at a different location (could be a FTP,CDN ).
I also started using [Zenhub](https://www.zenhub.com) which was suggested to me by Walter for handling issues and tracking progress and so far it's been working fine.
My repo is [jatindhankhar/aslo-v3](https://github.com/jatindhankhar/aslo-v3)


## Goals for next Week

For the next week I hope to contribute more to the code and disucss as well implement best practices wherever possible. 
As always, if you find any mistake, let me know and I'll fix it.
