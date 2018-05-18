---
title: Responsible Disclosure - Hacker Earth
categories: blog
excerpt: 'Taking over HackerEarth Sub Blog to RCE- HackerEarth'
tags: [programming,netsec]
comments: true
description: 'Taking over HackerEarth Sub Blog to RCE - HackerEarth'

---




## tl;dr 
Here is a tl;dr if you don't have the time to go through whole post.
Found an open un-configured worpdress sub blog and taking over it over to do Remote Code Execution and more.
Issue has been fixed by HackerEarth.

## How I found the issue

While doing subdomain enumeration for [https://hackerearth.com](https://hackerearth.com), I came across [http://demo-wordpress.hackerearth.com](http://demo-wordpress.hackerearth.com)
which had directory listing of many sub blogs. 
<img src="/images/disclosure-hackerearth/blog_listings.png">

Seeing directory listing pointed to possibily that something is mis configured, so I started opening every sub blog link and behold, there it was, an unconfigured wordpress instance, ready to be taken over, `innovation_from_recurit` blog was open and ready to be configured.
Here is how it looked like 

<img src="/images/disclosure-hackerearth/wp_installation.png">

I followed the installation and was able to gain admin rights to the following sub blog. 

<img src="/images/disclosure-hackerearth/wp_installed.png">

Now, this in itself is an issue worth reporting but exploring it within bounds never hurts and helps make the report more worthy.
One further exploration, it turns out, wordpress is used as landing pages for new hackathons. So Hackerearth uses Wordpress extensively and there is always a possiblity that some one re-uses the same password, across many systems.
Now I needed to explore the system without shell access, thankfully, wordpress comes with lots of useful plugins that can be installed, usually they provide a convenient way to perform common tasks. 


To access Database, I used **Adminer** (which sounds like a malware to mine crypto currency :sweat_smile:) and I was able to explore database for other sub blogs as well, since they all were running on the same system.


<img src="/images/disclosure-hackerearth/adminer_panel.png">

Accessing proper table, all the information, including user_names,email address and hashed password were accessible. 
Although I couldn't recover the original passwords, but it was easy to reset the password of any user and post content on behalf of that user.


<img src="/images/disclosure-hackerearth/adminer_blog_users.png">



Using **File Manager Advanced** to explore file system 

<img src="/images/disclosure-hackerearth/file_explorer.png">

Using **Developer Tools** to get system info 

<img src="/images/disclosure-hackerearth/developer_tools.png">

and finally using **WpTerm** to execute some commands (commands that require a proper tty interface don't work), which can be further used to inject backdoors.

<img src="/images/disclosure-hackerearth/rce.png">




## Bounty

Although HackerEarth have a bug bounty program [https://www.hackerearth.com/docs/wiki/program/bounty/](https://www.hackerearth.com/docs/wiki/program/bounty/), they are not providing rewards yet :neutral_face:	

Instead, I will be a getting a cool T-shirt :smile: .

Aiming to have a new tech Tee for each day of a month :sweat_smile: .

## Timeline

* 2 May, 2018 [3:18 PM IST] - Contacted the Customer Support where to report the issue 
* 2 May, 2018 [3:21 PM IST] - Response from customer support to sent the details.
* 2 May, 2018 [3:33 PM IST] - Sent the details
* 4 May, 2018 - Asked for an Update. Said they are looking into it
* 8 May, 2018 - Response from concerened Team, regarding the fix. Fix was partial only listing was removed, blog was still there. Told them so
* 9 May, 2018 - They removed the blog but forgot to reply back (?)
* 10 May, 2018 - Asked to disclose the report after the fix.
* 11 May, 2018 [3:21 PM IST] - Double Confirmed the fix.
* 11 May, 2018 [7:12 PM IST] - HackerEarth Rewarded T-Shirt as bounty :)
* 15 May, 2018 [20:30 PM IST] - Shared the Disclosure draft with HackerEarth
* 18 May, 2018 - Draft approved and post Published.

## Thanks 

I contacted HackerEarth support and they were quite fast,helpful and supportive during the whole process :+1:.








