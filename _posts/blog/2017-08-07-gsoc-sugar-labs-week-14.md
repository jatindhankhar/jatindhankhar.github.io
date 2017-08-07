---
title: GSOC Sugar Labs - Week 14
layout: post
categories: blog/
excerpt: 'GSOC Suar Labs - Week 14'
tags: [programming,sugar labs,gsoc]
comments: true
description: 'GSOC Suar Labs - Week 14 '

---
This is the fourteenth post in the series of my weekly GSOC Sugar Labs, where I summarize my week of working with [Sugar Labs](https://www.sugarlabs.org) under GSOC.

## Weekly Journal 
This week Samuel and I worked on the frontend and tried to achieve a pleasant UI with a good UX(at least that's what we think :smile:)
Also Samuel fixed the i18n by adding support for variants of english (en-IN,en-GB) and refactoring the whole i18n branch.


### Refactoring and fixing the i18n support
[Samuel did all of the work of refactoring and fixing up my mistakes on the i18n branch](https://github.com/jatindhankhar/aslo-v3/commit/2010b87e9ecfeae0d7c0ddf3a72cc0895d02c9f9).
Which included moving the logic outside the `wsgi.py` and moving pagination away from the `views` and fixing . The way i18n is setup we handle each locale differently, unless it's `en` (English), then we re-direct all variants of it (`en-IN` Indian English, for instance to ) English `en`). However we treat `en-US` and `en-GB` differently, since they do all have translations in `po` files, although majority of them are in `en-US`. 

This is how redirection is done.
{% highlight python %}
def set_lang_redirect(app):
     def get_language():
         # Detect lang based on Accept-Language header
         langs = request.accept_languages
         if not langs:
             return 'en'
         langs = [lang.replace('-', '_') for lang in langs.values()]
         lang = langs[0]
         # treat 'en_*' languages as 'en' except for en_US and en_GB
         if 'en_' in lang and lang not in ['en_US', 'en_GB']:
             lang = 'en'
 
         return lang
 
     @app.route('/')
     def lang_redirect():
         lang_code = get_language()
         return redirect('/' + lang_code + request.full_path)
{% endhighlight %}


### UI
UI changing was most challenging part for me. Earlier we were using Bootstrap along with [bootstrap-material-design](https://github.com/FezVrasta/bootstrap-material-design) but now we are using a mix of [MdBootstrap](https://mdbootstrap.com/) and [port of Airspace theme by Themefisher](https://github.com/luminousrubyist/airspace-jekyll). Some parts like Navbar and Footer were taken from the [SugarLabs Website Redesign project](https://github.com/geekrypter/sugarLabsWebsiteRedesign). (Thanks to Pericherla Seetaramaraju)
Here are some screenshots of the iterations. You can also [experience the new UI here](http://aslo.jatindhankhar.in:5000/en/)


**Initial UI**
<img src="/images/gsoc-week-14/old_aslo.png" alt="Old Aslo">



**New UI**
<img src="/images/gsoc-week-14/new_aslo.png" alt="New Aslo">


**Old Aslo Detail Page**
<img src="/images/gsoc-week-14/old_aslo_detail.png" alt="Old Aslo Detail">


**New Aslo Detail Page**
<img src="/images/gsoc-week-14/new_aslo_detail.png" alt="New Aslo Detail">


One thing was worked on the was developers section. Earlier we used to show Developers in a vertical list, which extended the page length in some cases.


**Old Developer List**
<img src="/images/gsoc-week-14/old_developer_list.png" alt="Old Developer List">

Then I thought of replacing the developer card with pills.


**Old Developer Pill List**
<img src="/images/gsoc-week-14/old_developer_pill_list.png" alt="Old Developer Pill List">

Later list was removed and developer pills were placed inside `sections`.

<img src="/images/gsoc-week-14/developer_pill_section.png" alt="Developer Pill Section">


**Big Picture with Pills**
<img src="/images/gsoc-week-14/aslo_detail_pill.png" alt="Aslo detail with Developer Pill Section">


which was finally replaced by horizontal scroll list/carousel with pill as items. Implementation was inspired by  [https://bootsnipp.com/snippets/featured/carousel-product-cart-slider](https://bootsnipp.com/snippets/featured/carousel-product-cart-slider). 


**Developer Carousel**
<img src="/images/gsoc-week-14/developer_carousel.png" alt="Developer Carousel">

Developer section was not working on small screens so now it gets hidden on small (`sm`) and extra small screens (`xs`) so as to ensure smooth UX. 
Categories were made to look like tags.

## Goals for Next Week
Now that UI is almost in place. This week I intend to focus on testing and documentation. Can't wait to release a fully fledged version of aslo-v3 ( which will be happening very soon )
If you find any typos,mistakes or any other inconsistencies, let me know and I'll fix them.