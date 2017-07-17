---
title: GSOC Sugar Labs - Week 11
layout: post
categories: blog/
excerpt: 'GSOC Suar Labs - Week 11'
tags: [programming,sugar labs,gsoc]
comments: true
description: 'GSOC Suar Labs - Week 11 '

---
This is the tenth post in the series of my weekly GSOC Sugar Labs, where I summarize my week of working with [Sugar Labs](https://www.sugarlabs.org) under GSOC.

## Weekly Journal 
This week Samuel and I refined the image management module, added a communication bot and frontend. 

First major thing that was on the list was bot communication, add first we were thinking to use [Github Statuses](https://developer.github.com/v3/repos/statuses/) but they are not that effective while communicating status back to developer. One big advantage of using comments is that developer will receive both email and notification, so it helps us make sure that news get delivered. To comment and for other future purposes we made a special bot named [aslo-v3-bot](https://github.com/aslo-v3-bot) and we uses it's Oauth token to comment on the release tag. When a release is made github sends the tag number used but not the commit :sad:, so to find the commit (so we can comment on that) we retrieve a list of tags and then find the target commit. 

{% highlight python %}
def find_tag_commit(repo_name, tag_name):
    g = auth()
    tags = g.get_repo(repo_name).get_tags()
    for tag in tags:
        if tag.name == tag_name:
            return tag.commit

    return None
{% endhighlight %}

Once we have the commit we can comment on the commit. We do it two times once build starts and once when build finishes along with the status (success or failure), along with the details.

{% highlight python %}
def release_process(self, gh_json):
    try:
        handle_release(gh_json)
    except (ReleaseError, BuildProcessError) as error:
        logger.exception("Error in activity release process!")
        tag_commit = find_tag_commit(
            gh_json['repository']['full_name'], gh_json['release']['tag_name'])
        comment_on_commit(
            tag_commit, "Build Failed :x: Details:  {}".format(error))
    else:
        logger.info('Activity release process has finished successfully!')
        tag_commit = find_tag_commit(
            gh_json['repository']['full_name'], gh_json['release']['tag_name'])
        comment_on_commit(tag_commit, "Build Passed :white_check_mark:")
{% endhighlight %}

Here are two such examples, one for [successful Build](https://github.com/sugarlabs-test/activity-turtleart-gtk3/commit/4469d77e9e3a173325ca1e647c3b3d4365b91873#commitcomment-23072564)

<img src="/images/gsoc-week-11/success.png" alt="Success Build">

and other for a [failed build](https://github.com/sugarlabs-test/browse-activity/commit/0a927969bade8bfd07d70e2c930351323f6b3fa2#commitcomment-23072507)


<img src="/images/gsoc-week-11/failure.png" alt="Failed Build">



Samuel pulled an all nighter :night: to fix the screenshots and implement the Model Service architecture, here are [some][http://gorodinski.com/blog/2012/04/14/services-in-domain-driven-design-ddd/] [useful links](https://lostechies.com/jimmybogard/2008/08/21/services-in-domain-driven-design/) on the methodology.

Frontend is finally taking up shape. I am using [bootstrap-material-design](https://github.com/FezVrasta/bootstrap-material-design). 
A demo [can be viewed here ] (http://aslo.jatindhankhar.in:5000/).
Each version of a activity gets an unique url which is following format `bundle_id/activity_version`
In case demo was inaccessible here are the screenshots

<img src="/images/gsoc-how-aslo-works/index.png" alt="Index Page">

<img src="/images/gsoc-how-aslo-works/detail.png" alt="Detail Page">

## Goals for Next Week
My plans for next week is to focus on Frontend and make a beautiful and minimal UI. Focus on testing and accessibility.

