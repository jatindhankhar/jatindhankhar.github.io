---
layout: post
title: "Jekyll's permalink and baseurl issue "
modified:
categories: blog/
excerpt: "Jekyll's permalink and baseurl don't play nice with each other and how to fix it. It all started with my new webcomic section"
tags: [programming,algorithms,c++,ruby]
comments: true
description: Jekyll's permalink and baseurl don't play nice with each other and how to fix it
---

It all started with my new [webcomic](/comics) section. I always wanted to have my own webcomic section and decided to build it using jekyll as it already powers my blog and I am familiar with it. Going by the docs, I decided to set it up as an independent jekyll blog to make things simple. Initially I decided to make  a theme of my own but then settled for [sudo-jekyll](https://github.com/oneohthree/sudo-jekyll) until mine is good enough to use.

# Plan


Since I decided to go for dedicated jekyll configuration for comics section, I used <code>baseurl</code>option which separates the jekyll site from the rest of the domain and ties to it's stipulated sub-directory, simple. No, I was wrong, sudo-jekyll wasn't considering baseurl so I had to edit layouts to replace <code>site.url</code> to <code>site.baseurl</code>which wasn't difficult and I thought this is it, but was it ?
Plus, I wanted my comic structure to be simple and having url's as following <code> /comics/0-n/ </code>and this was easy with jekyll's <code>permalink</code> option. So I setup my permalink option to be <code>/:title/</code> where title would be comic numbers

# Gotcha

No at all things go by the plan and I discovered that whenever I used permalink as <code>/:title</code>, individual comic entry were mapped to <code>https://jatindhankhar.in/0/</code> instead of the expected <code><code>https://jatindhankhar.in/comics/0/</code></code>. Confused, I changed permalink to add comics subdirectory <code>/comics/:title/</code> but I resulted into comics mapped to <code>/comics/comics/n/</code> and due to caching at CDN level I was getting correct results for few minutes and after purging old cache I got 404's.
tldr; Permalink was not obeying baseurl settings

# Fix

After seeing dozens of 404's which could have been avoided by locally testing the site before deploying (note to self, test site locally before deploying) and half hour of googling and reading blogs, I reached the [Jekyll documentation specific to github pages](http://jekyllrb.com/docs/github-pages/) which discussed the issue of permalink and baseurl and fix was to use <code> { { site.baseurl } } { { post.url } } </code> in post templates. I applied the trick and it was done. So whenever doing baseurl and perma-linking add following code to the target liquid template
<code> { { site.baseurl } } { { post.url } } </code> <b> without any slashes and spaces</b> in between
