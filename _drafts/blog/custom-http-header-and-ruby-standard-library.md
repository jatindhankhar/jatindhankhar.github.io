---
title: Custom Http Header and Ruby Standard Library
categories: programming
excerpt: 'Adventures with using custom http header and how ruby''s NET::HTTP library
  parses it '
tags:
- ruby
- programming
comments: true
description: 'Adventures with using custom http header and how ruby''s NET::HTTP library
  parses it '

---
# The problem

One day at work I got an esclation from one of the third party vendors that all the api calls to them were silently getting rejected on their end. They provided the explanation that one of the http header that was used to supplying the api key itself , (let's call it  `API-KEY`, for this post)  was being sent incorrectly.  

They wanted the header to be lower case like `api-key` but we started standing it in upper case `API-KEY`. This should not happen since we had a monkey patch to handle this situation.