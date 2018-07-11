---
title: Responsible Disclosure - Codementor
categories: blog
excerpt: ''
tags: [programming,netsec]
comments: true
description: 'Finding Open Redirect, XSS and Multiple subdomain takeover'

---




## tl;dr 
Here is a tl;dr if you don't have the time to go through whole post.
Found an open redirect exploit in Codementor's auth flow, which doubled as a XSS exploit. 
Also, an almost subdomain takover.


## The Bug
While logging into Codementor I noticed a `from` and `to` parameter in the auth-flow url. It piqued my interest,since redirect paramters are notorious for open redirect exploits.

### 1. Open Re-direct

To test for open redirect bug, obvious place was to look at the `to` parameter since it controls where the user will be redirect. While testing my hopes were not high as it's rare to find open redirects, but you never know until you try. 
I crafted following url
 `https://www.codementor.io/auth-redirect?from=https://www.codementor.io/login&to=https://www.google.com/` to redirect the user to `google.com` and, voila, it worked. After seeing many writeups on open redirect, I was happy to finally find and test one in the open (like a pokemon :smile:).

### 2. XSS

Whenever there is open redirect, there is a tiny possiblity that it can be converted into XSS. 
By subsituting the url in the `to` parameter with a javascript snippet we have the capability to execute arbitrarty code on the user's machine.

Same auth flow can be exploited as XSS, if to parameter is replaced by javascript code

`https://www.codementor.io/auth-redirect?from=https://www.codementor.io/login&to=javascript:alert(%271%27);`

will execute `alert(1)` on the user's machine. Of course it can be used to do cookie stealing and more mailcious tasks.

<img src="/images/disclosure-codementor/xss.png">


POC of the above issues

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/ejA0JGsfSsw?rel=0&amp;ecver=1" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

### 3. Multiple Subdomain Takeover

#### This isn't clear whether it was real issue or not. I believe it was, Codementor thinks it wasn't. Maybe I am wrong. If you know better, let me know. 

Looking at DNS entries it is evident that majority of Codementor infrastructure runs on Heroku. 
While doing sub domain enumeration, I came across some domains that point to heroku instances that were available, so I grabbed them. 

Some of the sub domains were [community-api.codementor.io](https://community-api.codementor.io), [hire-api.codmentor.io](https://hire-api.codmentor.io), [event-api.codementor.io](https://event-api.codementor.io) and most of them pointed to an available domain on heroku `kochi-8109.herokussl.com`. Since the domain was available, I grabbed one for myself.
Domain I grabbed was `kochi-8109.herokuapp.com`.
However to confirm the theory I had to add cname entries to my domain which required having a paid plan which I tried to subsribe but Heroku couldn't recognize/process my Debit card :disappointed: .

<img src="/images/disclosure-codementor/heroku.png">
So I reported the issue as it is.

Before reporting the issue Codementor's DNS entries were as following. Most of them pointed to `****.herokussl.com` 

{% highlight bash %}
host community-api.codementor.io                                   ✘ 130 
community-api.codementor.io is an alias for kochi-8109.herokussl.com.
kochi-8109.herokussl.com is an alias for elb049633-259037725.us-east-1.elb.amazonaws.com.
elb049633-259037725.us-east-1.elb.amazonaws.com has address 174.129.236.6
elb049633-259037725.us-east-1.elb.amazonaws.com has address 23.23.229.39
elb049633-259037725.us-east-1.elb.amazonaws.com has address 23.23.232.248

 host event-api.codementor.io
event-api.codementor.io is an alias for kochi-8109.herokussl.com.
kochi-8109.herokussl.com is an alias for elb049633-259037725.us-east-1.elb.amazonaws.com.
elb049633-259037725.us-east-1.elb.amazonaws.com has address 23.23.229.39
elb049633-259037725.us-east-1.elb.amazonaws.com has address 174.129.236.6
elb049633-259037725.us-east-1.elb.amazonaws.com has address 23.23.232.248

host hire-api.codementor.io
hire-api.codementor.io is an alias for kochi-8109.herokussl.com.
kochi-8109.herokussl.com is an alias for elb049633-259037725.us-east-1.elb.amazonaws.com.
elb049633-259037725.us-east-1.elb.amazonaws.com has address 23.23.232.248
elb049633-259037725.us-east-1.elb.amazonaws.com has address 23.23.229.39
elb049633-259037725.us-east-1.elb.amazonaws.com has address 174.129.236.6
{% endhighlight %}


After reporting the issue they abruptly changed the DNS entries. Most of them now point to `****.herokudns.com`

{% highlight bash %}
 host community-api.codementor.io
community-api.codementor.io is an alias for community-api.codementor.io.herokudns.com.
community-api.codementor.io.herokudns.com has address 52.203.53.176
community-api.codementor.io.herokudns.com has address 52.21.108.248
community-api.codementor.io.herokudns.com has address 52.207.5.158
community-api.codementor.io.herokudns.com has address 52.207.39.76
community-api.codementor.io.herokudns.com has address 52.22.127.224
community-api.codementor.io.herokudns.com has address 52.55.191.55
community-api.codementor.io.herokudns.com has address 52.22.2.149
community-api.codementor.io.herokudns.com has address 52.7.126.198
host hire-api.codementor.io
hire-api.codementor.io is an alias for hire-api.codementor.io.herokudns.com.
hire-api.codementor.io.herokudns.com has address 52.207.5.158
hire-api.codementor.io.herokudns.com has address 52.22.2.149
hire-api.codementor.io.herokudns.com has address 52.22.127.224
hire-api.codementor.io.herokudns.com has address 52.23.126.223
hire-api.codementor.io.herokudns.com has address 52.22.213.157
hire-api.codementor.io.herokudns.com has address 52.21.108.248
hire-api.codementor.io.herokudns.com has address 52.3.63.2
hire-api.codementor.io.herokudns.com has address 52.4.95.48


host event-api.codementor.io
event-api.codementor.io is an alias for event-api.codementor.io.herokudns.com.
event-api.codementor.io.herokudns.com has address 52.44.53.64
event-api.codementor.io.herokudns.com has address 52.207.5.158
event-api.codementor.io.herokudns.com has address 52.44.230.61
event-api.codementor.io.herokudns.com has address 52.55.191.55
event-api.codementor.io.herokudns.com has address 52.203.53.176
event-api.codementor.io.herokudns.com has address 52.5.182.176
event-api.codementor.io.herokudns.com has address 52.207.39.76
event-api.codementor.io.herokudns.com has address 52.7.126.198
{% endhighlight %}

After reporting the issue, this was Codementor's reply 

<blockquote>
Hi Jatin,

Sorry for the late reply. One of the mentors on our platform reminded us of these vulnerability. We were able to fix the issues with Open Re-direct and XSS. However, our website are not pointed to domains like xxx.herokuapp.com, so there won't be a problem if that domain is experiencing a takeover. 

Since when you sent in the report, these issues weren't fixed yet or reported publicly by us. We are offering a total of $50, do you have a PayPal account that we can transfer to? 
</blockquote>

Codementor acknowledged the first two issues but didn't acknowledge the last issue (sub domain takeover) as a valid one, even though they changed the DNS entries (why change the entries if it's okay :confused: ?)

I asked them to re-consider the third report and provided the proof ( sudden change in the DNS entries).
Their reply was 

<blockquote>
Hi Jatin,

Sorry for the confusion, but the change was made for operation purposes, it was a switch for the services they provided. 

The third issue you have reported actually was pointing to the domain herokussl.com, not the herokuapp.com you reserved. Since we are only considering the first two issues, that's why we are offering a total of $50. Thanks for understanding
</blockquote>

AFAIK, had I been able to subscribe to paid plan and add cname, I could have redirected traffic from `**.herokussl.com` or maybe Codementor team was right. If you know what went wrong, let me know in the comments below (damn, this sounds like a fellow Youtuber :laugh:)


## Bounty
I received a bounty of $50. This may not sound like a lot and I negotiated with Codementor.

But this was my first paid reward apart from swag, so yeah, not much to complain about.

## Thanks
Thanks to Codementor for fixing the bugs and permission to disclose the issues.