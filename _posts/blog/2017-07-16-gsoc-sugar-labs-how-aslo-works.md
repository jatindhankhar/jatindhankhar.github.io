---
title: GSOC Sugar Labs - How ASLO Works 
layout: post
categories: blog/
excerpt: 'GSOC Suar Labs - How ASLO Works'
tags: [programming,sugar labs,gsoc]
comments: true
description: 'GSOC Suar Labs - How ASLO Works '

---
This post is a break from my weekly GSOC Weekly Logs. This one is special, this one is dedicated to  how ASLO works and how developers will use it.


### Code and Technologies
Code is available on [jatindhankhar/aslo-v3](https://github.com/jatindhankhar/aslo-v3). 
Here is the Tech Stack
* Flask as backend framework;
* Celery for background tasks;
* Gunicorn as a WSGI server (which will be proxied by nginx in production);
* MongoDB (for internal database);
* Docker (for isolated builds);
* Github (for hosting the repos, release triggered webhooks, issue tracking);
* BootStrap Material Design for frontend.
## How ASLO works


With technologies introduced above, following we'll explain ASLO backend and frontend. 

### Backend

ASLO tries to leverage Github as a platform and avoid re-inventing wheel wherever possible. Code is hosted in a Github organization which is sugarlabs-test for now but eventually will be sugarlabs-activities. Whenever a developer reaches a milestone, he/she releases a new version on Github, which [triggers](https://developer.github.com/webhooks/) [a release webhook](https://developer.github.com/webhooks/) with event type being release.

GitHub release interface allows us to create a release for a specific tag (or create a new tag) and insert releases notes. Aside from that, it also allows us to attach binaries. Leveraging this, developers can also attach their own bundle when doing a release.

How does it works?

1. Developer releases a new activity version.
2. GitHub triggers a release webhook which sends a POST request with JSON payload to the ASLO app endpoint ([domain.tld]/api/hook).
3. ASLO receives the HTTP request with JSON payload and verifies required headers are present like for instance X-Hub-Signature which contains the hash signature in order to verify payload is actually coming from GitHub.
4. if Signature is valid, then ASLO parses the JSON payload and verifies if binaries have been attached in release. If binaries have been attached, then ASLO searches for valid .xo file. In case the .xo file is valid, then bundle won't be built by ASLO in release process.

Hence, we can say we have two kind of releases: source releases and assets releases. A source release is a release that doesn’t contain pre-bundled .xo file while an asset release contains prebuilt .xo file, giving developer the freedom to package their own .xo. 

In a source release, ASLO locally builds the .xo inside a Docker container while for an asset release just uses the pre-built .xo file attached by the developer. Let’s get a brief overview of building steps of each release.

#### Communication 
To communicate the status back to developer we use a Github Bot, ours is named `aslo-v3-bot`, say hello to it here [aslo-v3-bot](https://github.com/aslo-v3-bot). It  communicates by commenting on the commit through which tag and release was made. 
To comment on repos it needs to be member of that particular organization. 

Here are two such examples, one for [successful Build](https://github.com/sugarlabs-test/activity-turtleart-gtk3/commit/4469d77e9e3a173325ca1e647c3b3d4365b91873#commitcomment-23072564)

<img src="/images/gsoc-how-aslo-works/success.png" alt="Success Build">

and other for a [failed build](https://github.com/sugarlabs-test/browse-activity/commit/0a927969bade8bfd07d70e2c930351323f6b3fa2#commitcomment-23072507)


<img src="/images/gsoc-how-aslo-works/failure.png" alt="Failed Build">

It comments two times, when a build has started and other time when build is finished along with the build status. Comments are nice since, Github aslo informs the developers associated with the project. We are thinking to incorporate build status directly into the Github UI but that would require developers to use Pull Requests. 
To catch more errors before sending payload to ASLO, we propose to use [Travis CI](https://travis-ci.org) for each organization repo. 


##### Database
Backend was designed in mind to support multiple release, so users can download old release of an activity. 
There are two main collections `activity` and `releases`
<img src="/images/gsoc-how-aslo-works/activity_schema.png" alt="Activity Schema">

<img src="/images/gsoc-how-aslo-works/release_schema.png" alt="Release Schema">

By default new version will be shown but user can click on More Release button to discover old releases.
So each activity points to an array of `ReferenceIDs` which store old releases.


### FrontEnd 
For frontend we decided to use [bootstrap-material-design](https://github.com/FezVrasta/bootstrap-material-design)

A demo [can be viewed here ](http://aslo.jatindhankhar.in:5000/) 

Frontend is in (infancy `frontend` branch). For now  we show activity logo , developers who made that activity, a summary of the activity, categories (which we intend to use as tags) and meta data information like GTK version, license.
Each version of a activity gets an unique url which is following format `bundle_id/activity_version`
In case demo was inaccessible here are the screenshots

<img src="/images/gsoc-how-aslo-works/index.png" alt="Index Page">

<img src="/images/gsoc-how-aslo-works/detail.png" alt="Detail Page">
**


### How developers will use ASLO
Developers will keep working on their activities and whenever they think, they are ready to publish a new activity or a version of it, they will just release it and ASLO will handle the rest :smile: and inform about the Build status. 




#### Feedback and Suggestions

I would be glad to receive feedback and suggestions regarding ASLO, specially frontend. 
If you want to get involved don't be shy to Fork and sent a PR to [https://github.com/jatindhankhar/aslo-v3](https://github.com/jatindhankhar/aslo-v3)
If you want to reach me email me either at `[last_name].[first_name]@[gmail.com]` or `[first_name] @ [current_domain.com]`
I am specially thankful to Walter,Tony,James and Chris for their inputs and Samuel (my mentor) for helping me along the way and teaching me best practices.