---
title: 'Heisenbug:  A bug that happened in background only'
categories: blog
excerpt: Debugging an issue that occurred in background processes while running same
  query in rapid succession
tags:
- programming
- ruby
comments: true
description: Debugging an issue that occurred in background processes while running
  same query in rapid succession

---
# The bug

Few months back I ~~discovered~~ encountered a [Heisenbug](https://en.wikipedia.org/wiki/Heisenbug "heisenbug") (software bug that seems to disappear or alter its behavior when one attempts to study it.)

### How it all started ?

We were about to release a new settlement process for a set of users, one feature of the settlement process was keeping old weeks settled. So, even if any ledger was added to a settled week, system would automatically counter it to maintain the original amount for that week.

We tested it thoroughly and were about to release it next day post final desk check.

After I arrived at the office, I decided to test one more condition  to be really sure and that's when I encountered the heisenbug.

I tried to balance the amount of a user for a  settlement week.

I did so by adding two ledgers for a settled week but for the same day.

I noticed that the  processed week's amount is not correct and found counters ledgers are not correct.

In place of one ledger of X amount, system made two ledgers of X amount !

Thinking that I must have made some changes, I reset the development branch to last good commit, test the normal scenario on rails console and it worked, system balanced the amount.

I ran it again via the original flow, system failed to balance it.

![](https://en.meming.world/images/en/2/2c/Surprised_Pikachu_HD.jpg)

Trying to find the issue, I added debug logs, fired up the console, triggered the function that did all of the work, it worked just fine !

I made the script, ran it through rails runner mode, it worked again.

But when I used the main flow it didn't work. **Main flow was triggered inside a Sidekiq Worker**

Struggling to find the bug,  I  opened a  tmux window with split planes. Enabled activerecord query logs on both.

One with same flow in console and another containing sidekiq logs and stared at them a

Some minutes ago, Sajal from android team asked if rails does funny thing with threads and caching and I said don't be absurd, this is should not happen.

But then looking at the logs, it hit me.

Rails is indeed doing funny thing with sidekiq processes. Subsequent queries inside sidekiq was being cached.

![](/images/cache_issue.png)

# Why/How it happened ?

Console flow had following query

    # System computes difference 
    (2.1ms) SELECT SUM from table where date = 'DATE'
    # System balances first entry
    (2.1ms) SELECT SUM from table where date = 'DATE'
    # System balances second entry

while sidekiq flow only had real query and subsequent query was cached

    # System computes difference 
    (2.1ms) SELECT SUM from table where date = 'DATE'
    # System balances first entry
    # System computes difference
    CACHE (0ms) SELECT SUM from table where date = 'DATE'
    # System balances second entry
     

After balancing first entry, system computed the difference after adding second query but since query was same, rails didn't even fire the query, even though amount was affected and returned stale data and system tried to balance amount based on the stale data and reached in-consistent state. 

Now I knew what was happening I fixed the problem but let's find out how it happened. 

Googling 

# Fixing the bug