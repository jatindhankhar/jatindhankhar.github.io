---
title: GSOC Sugar Labs - Week 9
categories: blog
excerpt: 'GSOC Suar Labs - Week 9 '
tags: [programming,sugar labs,gsoc]
description: 'GSOC Suar Labs - Week 9 '

---
This is the ninth post in the series of my weekly GSOC Sugar Labs, where I summarize my week of working with [Sugar Labs](https://www.sugarlabs.org) under GSOC.

## Weekly Journal 

I am happy and more motivated by the fact that I passed the first review :smile:
This week I worked on the db layer and translations. Chris Leonard was kind enough to explain me the terminologies related to translation and i18n. Quoting a part of the reply from Chris 
<blockquote>
  Internationalization (i18n) = the process of preparing the code base
to be translated (e.g. tagging messages so that gettext tools will
extract a PO template (POT) file which is used to start the
translation process and later will insert messages from completed PO
files (technically MO files) when in use.  This step is always done by
the developer.

Localization (L10n) = the process of taking the POT file and turning
it into one Po file per language and then translating those PO files
and committing them to the repository.  This step is typically taken
care of by the Translation Team (and me) using our Pootle translation
server.
</blockquote>
Since we are aslo building a replacement NewASLO, it's sensible to reuse the work done by the translation for NewASLO and we will be re-using the strings for UI and Chris promised to commit the new changes whenver we need it.

For providing language specific strings per activity, we need to use `PO` files. Samuel and Chris pointed me in the right direction with that regard. I settled on [`polib`](http://polib.readthedocs.io/en/latest/quickstart.html) for handling Po files.
Struggling with it for a day I was able to write a usable parser/cralwer to get translated entries. 
So the process is really simple, we look for a `po` folder in activity, inside which we have several po files, one per language following the format `LANG_CODE.po`, so for each file we extract the lanugage code using `os.path.basename` and `os.path.splittext` then we iterate over each translate entry using polib's `polib.translated_entries` method, where each entry holds two things `msgid` which is the original un-translated text (usually in englis) and `msgstr` which is the translated entry for that particualr text in a language. To keep track of strings and their translated equivalent, we use a hash or `dict ` as we call it in python. 
Translations can be extracted by quering the dictionary in following manner.
`translations[language_code][target_string]`
So to access spanish version of "TurtleBlocks", we would do 
`translations["es"]["TurtleBlocks"]` and we would get "TortuBlocks" 

{% highlight python %}

from polib import pofile
from glob import glob
import os
from pprint import pprint


translations = {}
def get_language_code(filepath):
    basename = os.path.basename(filepath)
    return os.path.splitext(basename)[0]

# Replace it with the po file location

matched_files = glob("repos/activity-turtleart-gtk3/po/*.po")
po_files = list(map(pofile,matched_files))
language_codes = list(map(get_language_code,matched_files))

# Intialize the dictionary
for language_code in language_codes:
    translations[language_code] = {}

for po_file,language_code in zip(po_files,language_codes):
    for entry in po_file.translated_entries():
        #print(entry)
        translations[language_code][entry.msgid] = entry.msgstr


pprint(translations)
{% endhighlight %}
[Here is the gist link](https://gist.github.com/jatindhankhar/d450d86755a39909909c31cece65cc90) which contains code as well as the translation dump in json format.
Now next thing on the list was storing screenshots and icons. Samuel and I discussed on the possibility on storing images on file system which would require us to maintain dedicated paths for them, also backing up and migrating them would require extra work. We could have used [Mongo's GridFS](https://docs.mongodb.com/manual/core/gridfs/) and [mongoengine supports it as well](http://docs.mongoengine.org/guide/gridfs.html) but [it's not a good thing to do it for small objects (less than 16 MB) instead they can be inserted inside database using the binary type](https://docs.mongodb.com/manual/core/gridfs/#when-to-use-gridfs) , but screenshots can vary in size and we may have outlier where we cross 16 MB limit and `Gridfs` is not a good choice for most small sizes due to chunk overhead and extra database round trips, so easiest way was to offload the images to a third party and idea of hosting on Imgur came up , since it was aslo used in [`aslo-2`](https://github.com/sugarlabs/browse-activity/blob/master/activity/activity.info#L22)
. Plus Imgur have a nice official python library named [ImugrPython](https://github.com/Imgur/imgurpython) and [API is free to use for non-commercial projects](https://api.imgur.com/#freeusage)


## Goals for Next Week

Backend design is becoming more mature. I hope to finsih majority of the image storage and database work by the end of this week and hope to start frontend work by the mid of next week. If you find any mistakes/typos in this post, let me know and I'll fix it. 


