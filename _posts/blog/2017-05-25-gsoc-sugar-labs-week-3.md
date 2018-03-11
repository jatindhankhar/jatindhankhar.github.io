---
title: GSOC Sugar Labs - Week 3
categories: blog/
excerpt: 'GSOC Suar Labs - Week 3 '
tags: [programming,sugar labs,gsoc]
description: 'GSOC Suar Labs - Week 3 '

---

This is the third post in the series of my weekly GSOC Sugar Labs, where I summarize my week  of working with [Sugar Labs](https://www.sugarlabs.org) under GSOC.


## Weekly Journal

This week we discussed the possible features of ASLO, codenamed as ASLO v3. Walter instructed me to address some issues regarding ASLO. Here is a raw but slighlty modified excpert from the IRC chat before the acutal discussion

<blockquote>
 (1) auto sync between git and whatever we use to distribute/update apps that is at the heart of what Sam proposes
 (2) a place to browse activities, that could be a refresh or rewrite of ASLO
   James is advocating for the former, Tony for the latter
(3) a means of managing updates (this is the microformat, not necessarily tied to ASLO
(4) some mechanism for selecting activities for builds

</blockquote>

After which a Hangouts talk was scheduled which was converted to IRC talk due to issues with the voice ( my Laptop has single jack for audio and microphone but Headset had separate jacks :confused: ). Both Walter and Samuel were kind enough to understand the situation :smiley:
Also a more clear and promising ASLO proposal was released in the mailing list recently which advocates use of Github as database and management tool, making ASLO database less. I think this is good as it decreases the maintainance cost and Github API can be used to fetch release and Webhooks as signals/notifications. It was also decided to look at [Sam Today's version of ASLO (codenamed ASLO v2)](https://github.com/samdroid-apps/aslo), which uses Github extensively and is written in Flask. I managed to get the code running and host the code on digitalocean, learned few more things about docker. I reviewed the code which is divided in many folders and posted my findings on the mailing list, here is a part of it.

<blockquote>

So I cloned the SamToday's repo on digitalocean and ran only the web version and so far it works nicely. It can be accessed here <strong>[url removed for security reasons] </strong>.
<br>
I really like that it's written in Flask, makes it easier to navigate between files, since there are few files and directories to look at. I really don't understand the need of using Kafka. It looks it uses a small subset of bundles for demo, which can be viewed at <strong>[url removed for security reasons] </strong>.
<br>
But it doesn't use any database yet, not even Redis for  in-memory storage all the data is cached in memory of the running process.

<br>
Also base64 encodes the images.

<br>
Uses fully qualified package name of the activity to uniquely identify activity, which it becomes part of the url. Some UI  changes are needed (like showing activity icon on the detail url.
  <br>
I also liked the idea of using Docker to containerize the ASLO, making it easier to run and deploy the app.

</blockquote>

Then Samuel posted a detail review of ASLO on the mailing list the very next day. Here is a small and modified part of the it

<blockquote>
Hi guys!


I have review Sam's code and this is what I've found so far:


1. Initially, all activities shown in the web interface are read/parse from the JSON files uploaded to: https://github.com/samdroid-apps/sugar-activities. Using a repo as a activities registry has advantages and disadvantages.

<br>

2. The main web app spawns a thread which polls a kafka message queue (consumer) in order to get the new messages (GitHub payloads) inserted into the message queue by events sent to https://hook.sugarlabs.org/hook.

<br>
3. Let's review now the bot's part so we can understand the whole workflow. Inside repo we can find two folders: bot and bot-master.

<br>
Bot-master is a kafka consumer for the topic "org.sugarlabs.aslo-changes". This topic is different for the one used at [4] (org.sugarlabs.hook). Simplifying, every new topic is a new message bus. As a consumer, it needs to receive some message to process first.

<br>
<b>My conclusion</b>
<br>

After reviewing code, I still believe the proposal I've sent before  is the simplest way to manage this. Of course, maybe we can optimize some stuff along the way. Maybe we can add the chance to upload bundles file (XOs) manually as well, so we don't "remove" devs the chance to upload theirs XOs.


Reading SamP who wrote ASLO2, I'm still even more convinced (proosal) is the right way. He basically described this proposed solution in previous email: http://lists.sugarlabs.org/archive/iaep/2017-May/019761.html. Of course, ASLO2 gives us a lot of code we can reuse for ASLO3.
</blockquote>

Samuel did a great job of reviewing and explaining the code (Thanks, Samuel :+1:)

## Goals for next Week

30 May is the last day of community bonding and from next week Coding phase will begin. I am happy with the progress made, even more happy with the discussion going and creation of a new detailed proposal for ASLO3. I am thinking to use Flask. Hoping to meet the expectations of the people and myself. Let's see what happens :thinking:
<br>
As always, if you find any mistake, let me know and I'll fix it.
