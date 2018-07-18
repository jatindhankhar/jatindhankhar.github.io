---
title: Responsible Disclosure - HackerRank
categories: blog
excerpt: 'Reading other user`s  code submissions  - HackerEarth'
tags: [programming,netsec]
comments: true
description: 'Reading other user`s code submissions  - HackerEarth'

---




## tl;dr 
Here is a tl;dr if you don't have the time to go through whole post.

Found a way to read other user's submissions via Code Editor.

## How I found the issue

[HackerRank](https://www.hackerrank.com/) is a online programming paltform to learn,compete and find jobs. Like many online code platform, first thing to look for is the code editor and check if it can be exploited. 

I was able to do directory traversal and read some files and it was successful. I was able to read `passwd` files, read Android KeyStore files. Excited with my new findings, I contacted the HackerRank support and it turned out be a feature :neutral_face:

This was their response

<blockquote>
The files /etc/resolv.conf, /etc/passwd and few jar files and a C++ header file are provided with read-only access to the user. You are allowed to do file traversal and read files. Our execution environment allows those operations, with certain limitations.

We allow this access to all system files so that all code submissions can run correctly and can use standard library functions.

Certain files can be modified and are allowed because of certain use-cases. For example, if someone wants to store the code state in a file at one point and can read it back later at a different point of execution.

If you find a way to manipulate the system files or can read other user's submissions, then that's a valid vulnerability, and if you have discovered such an exploit, then it will be considered for the bounty program.

</blockquote>

Turns out, it was a feature and It was a false positive. I come around many false positives :sweat_smile:


However, If I can find a way to read other user submissions then it's a valid vulnerability and eligible for bounty, so in short,if I can read other's code then I am good to go :smile:


I was about to give up because I was tired, but I wanted to give it one more shot before throwing in the towel.

I tried to read logs, but there were none that could be accessed via code editor. If somehow I can read logs or list of files that are open for each users I can do something.
When a user submits a code, a random user on the machine is assinged, usually it's of the form `execution-user-xxxxx`
but if we check the working directory of users it's of the following form `/run-xxxxxxxxxxxx`

<img src="/images/disclosure-hackerrank/user_files_structure.png">

But I checked this after discovering another way to read the submissions.
I originally wanted to know what files were open during the code execution which can point me in the right direction.
So I tried the `lsof` command which returns **list** of **open files**

<img src="/images/disclosure-hackerrank/lsof_output.png">

and all the user submissions were on the root level 

<img src="/images/disclosure-hackerrank/root_structure.png">

Structure of a user directory is of following form 

```bash
/run-******************/:
compile.err
error00006.err
error00007.err
input
input00006.in
input00007.in
output00006.out
output00007.out
request.json
response_part.json
solution
solution.cc
```

**If we can read solution.*, we can find solutions that were submitted roughly around the same time period, plus reading input000xxx.in we can find out potential test cases to any problem and cheat our way through** :smiling_imp:


Let's read the the solutions submitted for `c` .

`less /run-*/solution.c`

<img src="/images/disclosure-hackerrank/c_solution.png">

what if we don't the know extension of the solution we can either do `less /run-*/solution.*` or just print the `request.json` as it contains solution submitted as well some other meta data.

`less /run-*/request.json`

<img src="/images/disclosure-hackerrank/json_response.png">


Sadly we cannot see `response.json` due to permission issues, otherwise we could have extracted more juicy info (?)

If we want to know hidden test cases for a problem, no problemo, just do `less input*.in`

When I had enough evidence that I could read user submissions, I submitted the same. They confirmed the same and asked me to explore further possibilites, if I can do manipulate files or link back each solution to the user. 

<blockquote>
We see that that the process id and the execution-user XXXX belongs to the sandbox environment the code is running in. Please explore further and see if you can get more other user info or exploit our code checker or hack into the server, we will gladly pass on the information to our team. 
</blockquote>

I tried out some things, looked for environment variables,deep directory traversal, causing kernel panic. 
There were some dangerous things like dumping a zip bomb which I didn't try, but hinted about all the possibility attacks in theory but there was nothing more I could think to attack. Hence I threw the towel.

This was HackerRanks' final response 

<blockquote>
As of now, we do not see a major vulnerability, but for all your effort we would like to show our appreciation by sending hackerrank T-Shirt your way
</blockquote>




## Bounty

Since I wasn't able to manipulate system files or link back a solution to a user, no bounty was rewarded. But HackerRank appreicated my effort and decided to send a cool T-shirt my way
So no reward but a cool T-shirt :smile: 

Gifts are always appreciated :smile:



## Timeline

* 9 May, 2018 [11:01 PM IST] - Contacted the Customer Support and reported the issue.
* 10 May, 2018 [2:54 AM IST] - Response from HackerRank rep.
* 10 May, 2018 [4:07 AM IST] - Response from HackerRank's Techincal Solutions Engineer. Initial report was false positive. Was told to look for a way to read user submissions
* 10 May, 2018 [9:33 PM IST] - Found a way to read user submissions. Reported the same
* 11 May, 2018 [11:14 AM IST] - Confirmation from HackerRank. Asked if I can explore further
* 11 May, 2018 [11:31 PM IST] - Coudln't find a way to manipulate files and tie back submission to a user.
* 10 May, 2018 - Asked to disclose the report after the fix.
* 11 May, 2018 [3:21 PM IST] - Double Confirmed the fix.
* 14 May, 2018 [11:31 AM IST] - Asked for an update.
* 14 May, 2018 [10:35 PM IST] - Report wasn't eligible for bounty, but got a T-shirt
* 15 May, 2018 [12:06 PM IST] - Asked to disclose the issue
* 16 May, 2018 [12:02 AM IST] - Recieved confirmation
* 16 May, 2018 [11:35 PM IST] - Shared the draft
* 18 July, 2018 [2:56 AM IST] - Draft approved
* 18 July, 2018 [2:16 PM IST] - Draft published

## Thanks 

I contacted HackerRank support and they were prompt,helpful and supportive during the whole process :+1:.
So far all the companies I have contacted were quite nice, even when my reports were invalid or false positives.








