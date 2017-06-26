---
title: GSOC Sugar Labs - Week 7 and 8
layout: post
categories: blog/
excerpt: 'GSOC Suar Labs - Week 7 and 8 '
tags: [programming,sugar labs,gsoc]
comments: true
description: 'GSOC Suar Labs - Week 7 and 8 '

---
This is the seventh and eighth post (combined into one) in the series of my weekly GSOC Sugar Labs, where I summarize my week of working with [Sugar Labs](https://www.sugarlabs.org) under GSOC.

## Weekly Journal 

In the following bi-weekly period I worked on the aslo-v3 along with Samuel. We worked on improving the aslo-v3, adding functionalities and fixing bugs. Since GSOC code review is approaching fast, it was very exciting and adrenaline driven experience. One important feature was to support both release with and without prebuilt bundles. It took some thinking to support both and we had to branch out build pipeline to support both cases. I created a [test orginzation named sugarlabs-test](https://github.com/sugarlabs-test) to develop aslo-v3 and debug issues. Github Webhooks was addded to workflow since it lets you to replay webhooks as many times you want and it helped debug issues with a particular release. To recieve locally I used [`ngrok`](http://ngrok.io/) and then settled on [`ultrahook`](http://www.ultrahook.com/) since ngrok provides namespace static url for free while to get a static url we have to switch to non-free model, but ultrahook only lets you receive POST events which ngrok supports all verbs and lets tunnel all traffic. Identification between a asset and non-asset release was simple. An asset release has an `asset` object in the json payload. Here is an example

{% highlight json %}
    "assets": [
      {
        "url": "https://api.github.com/repos/sugarlabs-test/4253-activity/releases/assets/4138906",
        "id": 4138906,
        "name": "FotoToon-10.xo",
        "label": null,
        "uploader": {
          "login": "jatindhankhar",
          "id": 2412081,
          "avatar_url": "https://avatars0.githubusercontent.com/u/2412081?v=3",
          "gravatar_id": "",
          "url": "https://api.github.com/users/jatindhankhar",
          "html_url": "https://github.com/jatindhankhar",
          "followers_url": "https://api.github.com/users/jatindhankhar/followers",
          "following_url": "https://api.github.com/users/jatindhankhar/following{/other_user}",
          "gists_url": "https://api.github.com/users/jatindhankhar/gists{/gist_id}",
          "starred_url": "https://api.github.com/users/jatindhankhar/starred{/owner}{/repo}",
          "subscriptions_url": "https://api.github.com/users/jatindhankhar/subscriptions",
          "organizations_url": "https://api.github.com/users/jatindhankhar/orgs",
          "repos_url": "https://api.github.com/users/jatindhankhar/repos",
          "events_url": "https://api.github.com/users/jatindhankhar/events{/privacy}",
          "received_events_url": "https://api.github.com/users/jatindhankhar/received_events",
          "type": "User",
          "site_admin": false
        },
        "content_type": "application/octet-stream",
        "state": "uploaded",
        "size": 270918,
        "download_count": 0,
        "created_at": "2017-06-20T09:42:13Z",
        "updated_at": "2017-06-20T09:42:53Z",
        "browser_download_url": "https://github.com/sugarlabs-test/4253-activity/releases/download/0.4/FotoToon-10.xo"
      },
      {
        "url": "https://api.github.com/repos/sugarlabs-test/4253-activity/releases/assets/4138907",
        "id": 4138907,
        "name": "TurtleBlocks-213.xo",
        "label": null,
        "uploader": {
          "login": "jatindhankhar",
          "id": 2412081,
          "avatar_url": "https://avatars0.githubusercontent.com/u/2412081?v=3",
          "gravatar_id": "",
          "url": "https://api.github.com/users/jatindhankhar",
          "html_url": "https://github.com/jatindhankhar",
          "followers_url": "https://api.github.com/users/jatindhankhar/followers",
          "following_url": "https://api.github.com/users/jatindhankhar/following{/other_user}",
          "gists_url": "https://api.github.com/users/jatindhankhar/gists{/gist_id}",
          "starred_url": "https://api.github.com/users/jatindhankhar/starred{/owner}{/repo}",
          "subscriptions_url": "https://api.github.com/users/jatindhankhar/subscriptions",
          "organizations_url": "https://api.github.com/users/jatindhankhar/orgs",
          "repos_url": "https://api.github.com/users/jatindhankhar/repos",
          "events_url": "https://api.github.com/users/jatindhankhar/events{/privacy}",
          "received_events_url": "https://api.github.com/users/jatindhankhar/received_events",
          "type": "User",
          "site_admin": false
        },
        "content_type": "application/octet-stream",
        "state": "uploaded",
        "size": 6020041,
        "download_count": 0,
        "created_at": "2017-06-20T09:42:13Z",
        "updated_at": "2017-06-20T09:42:52Z",
        "browser_download_url": "https://github.com/sugarlabs-test/4253-activity/releases/download/0.4/TurtleBlocks-213.xo"
      },
      {
        "url": "https://api.github.com/repos/sugarlabs-test/4253-activity/releases/assets/4138920",
        "id": 4138920,
        "name": "requirements.txt",
        "label": null,
        "uploader": {
          "login": "jatindhankhar",
          "id": 2412081,
          "avatar_url": "https://avatars0.githubusercontent.com/u/2412081?v=3",
          "gravatar_id": "",
          "url": "https://api.github.com/users/jatindhankhar",
          "html_url": "https://github.com/jatindhankhar",
          "followers_url": "https://api.github.com/users/jatindhankhar/followers",
          "following_url": "https://api.github.com/users/jatindhankhar/following{/other_user}",
          "gists_url": "https://api.github.com/users/jatindhankhar/gists{/gist_id}",
          "starred_url": "https://api.github.com/users/jatindhankhar/starred{/owner}{/repo}",
          "subscriptions_url": "https://api.github.com/users/jatindhankhar/subscriptions",
          "organizations_url": "https://api.github.com/users/jatindhankhar/orgs",
          "repos_url": "https://api.github.com/users/jatindhankhar/repos",
          "events_url": "https://api.github.com/users/jatindhankhar/events{/privacy}",
          "received_events_url": "https://api.github.com/users/jatindhankhar/received_events",
          "type": "User",
          "site_admin": false
        },
        "content_type": "text/plain",
        "state": "uploaded",
        "size": 51,
        "download_count": 0,
        "created_at": "2017-06-20T09:45:11Z",
        "updated_at": "2017-06-20T09:45:16Z",
        "browser_download_url": "https://github.com/sugarlabs-test/4253-activity/releases/download/0.4/requirements.txt"
      }
    ]
{% endhighlight %}

For asset release we download the bundle by checking asset for first valid .xo file (valid in the sense, it should have activity.info) then we store the bundle in appropriate directory, parse json and insert into it json.
A decision was to made support mutliple releases, aslo-v3 will store new and old releases, in case user want to download a particular version but by default only release will be shown and old releases can be accessed via a More Download/Advanced Options. To keep track of things, Samuel and I jotted down things in a shared Google Keep List. To collect release notes we will be using `[release][body]` and render them on Frontend, since it's just makrdown. 

Samuel was kind enough to refactor all the code and send a PR. Now most of the recent changes can be seen in [refactor branch](https://github.com/jatindhankhar/aslo-v3/tree/refactor). New refactored code is much more organized and easy to follow. It uses error handling by raising instead of relying on return values which reduced amount of code needed. Samuel aslo simplified the Docker.

{% highlight DockerFile %}
FROM fedora:22

RUN dnf install -y git gettext

RUN git clone --depth=1 https://github.com/sugarlabs/sugar-toolkit-gtk3; \
    mv sugar-toolkit-gtk3/src/sugar3 /usr/lib/python2.7/site-packages/sugar3; \
    rm -rf sugar-toolkit-gtk3

RUN rm -rf /usr/lib/python2.7/site-packages/sugar; \
    ln -s /usr/lib/python2.7/site-packages/sugar3 /usr/lib/python2.7/site-packages/sugar

VOLUME /activity
WORKDIR /activity

CMD python /activity/setup.py dist_xo --no-fail; \
    chmod 777 -R /activity/

{% endhighlight %}

and build process and I added/refactored rest of the code. We switched back to `fedora-22` since most of the activities are tested in Fedora (SOAS is also built upon Fedora) and it would save us from weird errors that may arise due to dependency on fedora on activity.
Samuel was also kind enough to give me a walk through and explain code via Google Hangouts :smile: .
Samuel aslo suggested me to talk to Chris Leonard and discuss the localization process with him and how [NewAslo](https://translate.sugarlabs.org/projects/NewAslo/) can be used for localization.
Also it was decided to invent a heuristic to detect Sugar version for which I reused the SamToday' code. It was also decided to use Mongoengine to provide some structure to DB specific code and ORM really does reduces the code. We also pondered upon the schema and discussed the one big documents vs many small/moderate collections, each for activity.
Samuel came up with following schema (still iterating over )
{% highlight json %}

# collection: activities

{
  _id: 'a1b234',
  bundle_id: 'org.laptop.TurtleArtActivity',
  license: 'MIT',
  icon: 'activity-turtleart', #gridfs, filepath, svg: is xml-based?
  name: {
    en-US: 'TurtleBlocks', # we need to look how to integrate with https://translate.sugarlabs.org/projects/NewAslo/
    es: 'TortuBlocks',
    ... (other languages)
  },
  summary: {
    en-US: 'A Logo-inspired turtle that draws colorful pictures with snap-together visual programming blocks',
    es: 'Una tortuga inspirada en Logo que hace dibujos coloridos junto con bloques complementarios de programaci\u00f3n visual',
    ... (other languages)
  },
  categories: {
    en-US: ['programming', 'art'],
    es: ['programaci\u00f3n', 'arte'],
    ... (other languages)
  },
  developers: [
    { name: 'Walter Bender', email: 'walter@sugarlabs.org', page: 'https://github.com/walterbender'}, 
    { name: 'Other dev', email: 'someemail@somedomain', page: 'https://github.com/user'}
    # check https://developer.github.com/v3/repos/collaborators/
    # ex: https://api.github.com/repos/walterbender/activity-turtleart-gtk2/collaborators
  ]
  last_release: {
    activity_version: 214,
    release_notes: 'text defined in GH release interface',
    min_sugar_version: 0.86,
    timestamp: 'time_release_was_done',
    xo_url: 'http://download.sugarlabs.org/activities3/TurtleBlocks-214.xo', # maybe organize differently?
    is_gtk3: false,
    is_web: false,
    has_old_toolbars: false,
    screenshots: { #should we simplify and store screenshots just for one language?
      en-US: ['http://some-url', '/local-in-server'],
      es: ['http://some-url', '/local-in-server']
    }
  },
  previous_releases: [ { id: '2b1c1'}, { id: '49cd3' } ] #yeap, we'll have a releases collection
}

# collection: releases (why? we only incur in more delay when more releases are requested)
{
	_id: '2b1c1',
    activity_version: 213,
    release_notes: 'text defined in GH release interface',
    min_sugar_version: 0.86,
    timestamp: 'time_release_was_done',
    xo_url: 'http://download.sugarlabs.org/activities3/TurtleBlocks-213.xo',
    is_gtk3: false,
    is_web: false,
    has_old_toolbars: false,
    screenshots: {
      en-US: ['http://some-url', '/local-in-server'],
      es: ['http://some-url', '/local-in-server']
    }
}
{% endhighlight %}

Qutoing my mentor Samuel
<blockquote>
I have had a conversation with Walter and we are thinking to point out to old url releases.
So this collection is right place! Probably we'll end up writing a script to fill in theses
releases documents. Big work!
Why? Because we believe the only way to kill aslov1 is to shut it down!

</blockquote>
A solid schema is needed if we we aim to serve users and replace old aslo.
Only challenge we are yet to decide is the use of Mongoengine's `DynamicDocument and EmbeddedDynamicDocument classes` for storing dynamic documents with dynamic keys, mainly for storing different translations of summary,name and changelogs.

## Goals for Next Week
I have GSOC code review this week, so I hope I pass the review (wish me luck ) so I can continue working on aslo-v3 and make it successful.


