---
title: GSOC Sugar Labs - Week 10
categories: blog/
excerpt: 'GSOC Suar Labs - Week 10'
tags: [programming,sugar labs,gsoc]
description: 'GSOC Suar Labs - Week 10 '

---
This is the tenth post in the series of my weekly GSOC Sugar Labs, where I summarize my week of working with [Sugar Labs](https://www.sugarlabs.org) under GSOC.

## Weekly Journal 
This week Samuel and I completed a major chunk of backend and transitioned into frontend. While working on Frontend I faced a weird error where celery was complaining something about duplicated nodes. I tried every thing only reboot was fixing the error.
<img src="/images/gsoc-week-10/celery_error.png" alt="Celery Error">
 Then I googled the issue and many people were complaining about the same issue. I asked Samuel and he asked me show the output of  `ps fauxwww ` which was new to me  since I have used the simple version of it `ps aux`, turns out `ps auxwww  ` is a very useful command which increases the width of ps output and gives a complete picture of process running. 
 Back to issue, issue was caused due to multiple celery instances running in the memory which resulted in duplicated nodes. 
 But issue itself was weird and I never encountered it before/after. Solution was to kill all celery instances and 
 `killall celery` solved it in single line.
 Last week it was decided to use Imgur for image hosting but turns out Imgur doesn't support SVG images :sad:
 Samuel fixed and tested edge case of the backend and refactored code into easily navigable modules. One of the important thing that was fixed was the add release bug (mine had a bug :bug:)

 {% highlight python %}
    def add_release(self, release):
        # first release
        if not self.latest_release and len(self.previous_releases) == 0:
            self.latest_release = release
            return

        if self.latest_release.activity_version >= release.activity_version:
            release.delete()
            raise me.ValidationError(
                'New activity release version {} is less or equal than the '
                'current version {}'
                 .format(
                    release.activity_version, 
                    self.latest_release.activity_version
                 )
            )
    
        self.previous_releases.append(self.latest_release)
        self.latest_release = release
 {% endhighlight %} 
 

I also learned about [rdb](https://stackoverflow.com/questions/12698212/how-to-debug-celery-django-tasks-running-locally-in-eclipse/36690646#36690646)
which really helps in debugging ongoing celery tasks.

For the frontend bootstrap was decided and after checking out some implementations I decided to go with the [bootstrap-material-design](https://github.com/FezVrasta/bootstrap-material-design). We were thinking to use Angular or React for client side rendering but since we need to support Sugar Laptops, we decided not to pursue it.
Here are some of the very rough early drafts of the frontend design :sweat_smile: .


<blockquote> Index Page </blockquote>
<img src="/images/gsoc-week-10/index.png" alt="Index Page">

<hr>

<blockquote> Detail Page </blockquote>
<img src="/images/gsoc-week-10/detail.png" alt="Detail Page">

Do send me suggestions regarding designs if any :smile:
## Goals for Next Week

Backend design is almost complete. I hope to finish a major part of frontend as well and release a public beta for community to try out. If you find any mistakes/typos in this post, let me know and I'll fix it. 


