---
title: 'Responsible Disclosure:  Breaking out of a Sandboxed Editor to perform RCE'
categories: blog
excerpt: 'Responsible Disclosure:  Breaking out of a Sandboxed Editor to perform RCE'
tags:
- netsec
- programming
comments: true
description: Breaking out of a Sandboxed Editor of an online platform to perform RCE.

---
# tl;dr 

Found a way to escape the sandboxed editor to perform Remote Code Execution leading which lead to ability to view AWS credentials, ssl certificate, passwd files.   
Pretty much owning the entire machine 

# Story - Finding the issue

If you are still here after reading the tl;dr, I guess you are here for the story ?   
So, let me give you one.   
  
While doing recon I found many sub-domains and ip addresses belonging to Hackerearth, one of them was [https://18.140.198.247/#/home/node/he-theia/sandbox](https://18.140.198.247/#/home/node/he-theia/sandbox) which was running an online ide built on top of vs-code named  [Theia IDE](https://theia-ide.org/ "https://theia-ide.org/").

At first glance, it looked pretty boring, after all it'a a IDE running in a browser (wait, that's normal since most of them are electron based :|) 
So, anyways, I played around with it for a while, the ultimated goal was to execute random code on the machine. But, they removed the terminal view from the IDE shortcuts and menu. 
So, I tried to "run" the code file but that opened was also not available.
Then poking around I tried "Run 