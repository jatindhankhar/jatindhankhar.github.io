---
title: GSOC Sugar Labs - Week 12
layout: post
categories: blog/
excerpt: 'GSOC Suar Labs - Week 12'
tags: [programming,sugar labs,gsoc]
comments: true
description: 'GSOC Suar Labs - Week 12 '

---
This is the twelfth post in the series of my weekly GSOC Sugar Labs, where I summarize my week of working with [Sugar Labs](https://www.sugarlabs.org) under GSOC.

## Weekly Journal 
This week Samuel and I focused on improving the build image and release build process. We added support for old activities (activities that can be built by old-toolkit), activities without po files and license extraction. On the Frontend side, screenshot support was added.

Most of the work done this week was inspired by the discussion made on the following thread [https://github.com/sugarlabs/sugar-toolkit-gtk3/issues/369](https://github.com/sugarlabs/sugar-toolkit-gtk3/issues/369). 
James suggested to warn the developer of missing attributes instead of enforcing them. Discussion in this thread helped us uncover some issues. After sugar-toolkit was updated with instructions on installing, it became easier to install them which lead to change to `Dockerfile` and support for old activities as well, here is the pull request [https://github.com/jatindhankhar/aslo-v3/pull/6](https://github.com/jatindhankhar/aslo-v3/pull/6) . 

{% highlight Dockerfile %}
FROM fedora:22

RUN dnf install -y git gettext sugar-toolkit-gtk3 sugar-toolkit

VOLUME /activity
WORKDIR /activity

CMD python /activity/setup.py dist_xo; \
    chmod 777 -R /activity/

{% endhighlight %}
Now aslo-v3 supports both activities, old (uses old toolkit) and new (uses new gtk3 toolkit) :hooray:. This was the easiest fix since loading appropriate toolkit is done by the python interpreter, since import of the toolkit is defined in the `setup.py`.
Only downside is we need to drop `--no-fail` from the build instruction which is supported only by new toolkit.
  
Next most of the activities specially web based activities were getting rejected because they didn't have any `po` files which are needed for i18n but not all developers target all audience, so it became necessary to support this as well. This required us to reduce the `MANDATORY_ATTRIBUTES` list, adding fallback/default values in the database models and relying on the assumed `en` values are being a dictionary.

{% highlight python %}
 if any(translations):
       metadata['i18n_name'] = i18n.translate_field(metadata['name'],
                                                 translations)
       metadata['i18n_summary'] = i18n.translate_field(metadata['summary'],
                                                    translations)
    else:
        metadata['i18n_name'] = {'en' : metadata['name']}
        if 'summary' not in metadata:
            metadata['i18n_summary'] = {'en' : "No summary"}
        else:
             metadata['i18n_summary'] = {'en' : metadata['summary']} 

{% endhighlight %}

License extraction was also added to the aslo, since it was the most sensitive one but ideally developer should define the proper license.
I write a script to analyze all the public repos of [https://github.com/sugar-activities](https://github.com/sugar-activities) and were few of them didn't have any license. Here is [the link to the GIST](https://gist.github.com/jatindhankhar/ab299b5f318695cfaf6d05fd9ccda058)

{% highlight python %}
from github import Github
from github.GithubException import GithubException
import configparser
cp = configparser.ConfigParser()

def check_metadata_license(repo):
    try:
       repo.get_file_contents('activity/activity.info')
    except GithubException as error:
       return False
    metadata_string = repo.get_file_contents('activity/activity.info').decoded_content
    try:
    	cp.read_string(metadata_string.decode())
    except Exception as error:
        return False
    return cp.has_option('Activity','License') or cp.has_option('Activity','license')

def check_root_license(repo):
    try:
       repo.get_file_contents('LICENSE')
    except GithubException as error:
       return False
    return True


file_root_license = open('has_root_license.txt','w')
file_metadata_license = open('has_metadata_license.txt','w')
file_both_license = open('has_both_license.txt','w')
file_no_license = open('has_no_license.txt','w')
file_overall_stats = open('stats.txt','w')

# Replace xxxx string with the User's Oauth token
g = Github("xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx") 

sugar_activities  = g.get_organization("sugar-activities") 

public_repos = sugar_activities.public_repos

count_license_in_metadata = 0
count_license_in_root_dir = 0
count_both = 0
count_no_license = 0

processed_repos = 0
for repo in sugar_activities.get_repos():
    meta_check = check_metadata_license(repo)
    root_check = check_root_license(repo)
    if meta_check:
       print(repo.clone_url,file=file_metadata_license)
       count_license_in_metadata  = count_license_in_metadata + 1
    if root_check:  
       print(repo.clone_url,file=file_root_license)
       count_license_in_root_dir = count_license_in_root_dir + 1
    if meta_check and root_check:
       print(repo.clone_url,file=file_both_license)
       count_both = count_both +1
    if not(meta_check) and not(root_check):
       print(repo.clone_url,file=file_no_license)
       count_no_license = count_no_license +1
    processed_repos = processed_repos + 1
    print(repo.clone_url)
    print(" Processed {}  out of {} repos".format(processed_repos,public_repos))
    print("Repos with MetaData License : {}".format(count_license_in_metadata))
    print("Repos with LICENSE File : {}".format(count_license_in_root_dir))
    print("Repos with License entry in file and metadata : {}".format(count_both))
    print("Repos with No License : {}".format(count_no_license))


print("Total repos check : {} ".format(public_repos),file=file_overall_stats)
print("Repos with MetaData License : {}".format(count_license_in_metadata),file=file_overall_stats)
print("Repos with LICENSE File : {}".format(count_license_in_root_dir),file=file_overall_stats)
print("Repos with License entry in file and metadata : {}".format(count_both),file=file_overall_stats)
print("Repos with No License : {}".format(count_no_license),file=file_overall_stats)

file_metadata_license.close()
file_root_license.close()
file_both_license.close()
file_no_license.close()
file_overall_stats.close()
Raw

{% endhighlight %}

{% highlight bash %}
Total repos check : 687 
Repos with MetaData License : 647
Repos with LICENSE File : 45
Repos with License entry in file and metadata : 45
Repos with No License : 40
{% endhighlight %}

Aslo first looks for license inside the `activity.info` file, if not found looks for `LICENSE` and `COPYING` file and extracts the information from the headers. 
In the Frontend side, screenshots were <strike>added </strike>, enabled in the jinja templates.
## Goals for Next Week
My plans for next week are 
1. Test core activities ([https://github.com/sugarlabs/sugar-build/blob/master/build/modules.json](https://github.com/sugarlabs/sugar-build/blob/master/build/modules.json) activities start at line 90 )
2. Add Search (Searching by bundle_id or activity name)
3. Improve UI (Add Github Repo Information, make it more beautiful)
4. i18n (Make language specific routes like `/en` or `es/` and translate the aslo UI :smile:)
5. Test the aslo build container with Fedora 26
