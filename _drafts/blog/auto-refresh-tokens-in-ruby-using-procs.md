---
title: 'Auto Refresh tokens in Ruby using Procs  '
categories: blog
excerpt: Auto refresh API tokens seamlessly in ruby using procs.
tags:
- ruby
- programming
comments: true
description: Auto refresh API tokens seamlessly in ruby using procs.

---
# Backstory ? 

Few months ago I had to integrate a third party service into our system.   
The third party in question was used in a dashboard to verify certain  details of users. 

So in an ideal world, the end user just clicks on verify and our system handles rest of the stuff (like it should). 
So the service had an api which can be used to verify those details (Obviously :smile:) But it used a refresh token which expired every 15 minutes. 

[Refresh tokens are good from security point of view](https://auth0.com/learn/refresh-tokens/) but using them in normal flow can be tricky sometimes. 


# How ?

So we had to auto refresh token after certain intervals without causing any failures for the end user and that sometimes could happen in middle of the actual verification call and we needed to gracefully retry that. 

