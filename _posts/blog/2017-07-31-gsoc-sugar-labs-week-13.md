---
title: GSOC Sugar Labs - Week 13
layout: post
categories: blog/
excerpt: 'GSOC Suar Labs - Week 13'
tags: [programming,sugar labs,gsoc]
comments: true
description: 'GSOC Suar Labs - Week 13 '

---
This is the thirteenth post in the series of my weekly GSOC Sugar Labs, where I summarize my week of working with [Sugar Labs](https://www.sugarlabs.org) under GSOC.

## Weekly Journal 
Good news, I passed the second evaluation :smile:
This week Samuel and I focused on adding more features to aslo.
We added
* Search (Activities can be searched and supports all languages) 
* Pagination (Index, Search, every user facing template is paginated)
* Testing Core Activities with Fedora 26 (Almost all core activities work with Fedora 26)


### Updating Docker Image
This was done on a suggestion by James Cameron, since older docker image used fedora 22 which was very old. 
[Here is the Pull Request](https://github.com/jatindhankhar/aslo-v3/pull/8). It was tested and worked with most of the valid activities.

### Search
Search is the most basic feature that users will need, given that there are many activities to search for. 
User should be able to navigate to activities they need as fast possible. Currently only name based search is available, search by `bundle_id` will be added later. Originally search was done against `en` or `en_US` locale, but now all locales are supported. If a user is using `hi` (Hindi) version of aslo, names will be matched against hindi name (if any).

{% highlight python %}
def search_by_activity_name(activity_name, lang_code):
    lang_filter_query = me.Q(**{'name__' + lang_code + '__exists': True})
    name_match_query = me.Q(
        **{'name__' + lang_code + '__icontains': activity_name})
    return Activity.get_all().filter(lang_filter_query and name_match_query)
{% endhighlight %}

Since language name can be dynamic as well and Mongoengine follows Djanog's way of handling query. I used `kwargs` to build up dynamic queries and Mongoengines [Query Tree](http://motorengine.readthedocs.io/en/latest/getting-and-querying.html#querying-with-q) `Q` for boolean logic. Search comparison looks for substring inside the name and is not case_sensitive (`icontains`). To ensure language specific result, we look if that activity is available in the stipulated language (`__exists`) . 

Here are some results 

##### Tamil Search
<img src="/images/gsoc-week-13/tamil_search.png" alt="Tamil Search">

<hr>

##### Hindi Search
<img src="/images/gsoc-week-13/hindi_search_t.png" alt="Hindi Search">


### Pagination 
Pagination was very much needed, else it will put load on both server (server will render all available activities) and user (browser will render all activities, including downloading icons).
For pagination I followed the following [gist](https://gist.github.com/wonderb0lt/10645080). 
Pagination was added to `access.py` since it's a metaclass inherited by others, which ensures all the others classes have pagination support and adding a pagination template that has been improved now. 

{% highlight python %}
 class Pagination:
    def __init__(self, items, page, items_per_page, total_items):
        self.items = items
        self.page = page
        self.total_items = total_items
        self.items_per_page = items_per_page
        self.num_pages = int(ceil(total_items / float(items_per_page)))

    @property
    def has_next(self):
        return self.page < self.num_pages

    @property
    def has_prev(self):
        return self.page > 1

    @property
    def next_page(self):
        return self.page + 1

    @property
    def prev_page(self):
        return self.page - 1
{% endhighlight %}

And views handle the paging, something like this

{% highlight python %}
@web.route('/', defaults={'page': 1})
@web.route('/page/<int:page>')
def index(page=1, items_per_page=10):
    return Activity.paginate(page=page)
{% endhighlight %}

Ofcourse, this is a simplified version.


### i18n Support
This was the most challenging and fun of all. Most of the activities already have translations supplied and stored in the database. First thing needed was prefix/separating urls based on the language code, something like `/hi` (for hindi), `/en` (for english) , `/es` for (spanish/espanol). Since we were alreading using [blueprints](http://flask.pocoo.org/docs/0.12/blueprints/), it was as easy as adding a `url_prefix` to the web blueprint.

{% highlight python %}
 web = Blueprint('web', __name__, template_folder='templates',
                static_folder='static',
                static_url_path='/web/static',
                url_prefix='/<lang_code>') 
{% endhighlight %}

and to handle the urls, [`urlprocessor`](http://flask.pocoo.org/docs/0.12/patterns/urlprocessors/) came to rescue. Now here we process the url for prefix and pop the value so that the application is not aware of the prefixes and routing map works normally. However we need to translate the content and for that we need to know the language code of each request, that's where we tie each user session with a language code, so that it can be retrieved at any point later. 

{% highlight python %}
@web.url_defaults
def add_language_code(endpoint, values):
    values.setdefault('lang_code', g.lang_code)


@web.url_value_preprocessor
def pull_lang_code(point, values):
    g.lang_code = values.pop('lang_code')
    # Tie user session to a particular language,
    # so it can be rtrieved when we pop the request values
    session['lang_code'] = g.lang_code

{% endhighlight %}

One challenge was to handle the `/` (root url). Now everything is url prefixed with the language code in the web blueprint. It becomes difficult to recognize language code with `/`, because there is no root route anymore and gives `404`. This was solved by having separate route rule outside the blueprint for capturing root requests and predicting the language code based on the user headers. So whenever a user visits the root url, we predict the language code, and redirect user to the a localized version and [Flask-babel](https://pythonhosted.org/Flask-Babel/) to predict user language for translation UI elements itself ( in the coming future).

{% highlight python %}
@babel.localeselector
def get_locale():
    # if a user is logged in, use the locale from the user settings
    user = getattr(g, 'user', None)
    if user is not None:
        print("Return use locale")
        return user.locale
    # otherwise try to guess the language from the user accept
    # header the browser transmits.  We support de/fr/en/es/hi in this
    # example.  The best match wins.
    return request.accept_languages.best_match(['de', 'hi', 'fr', 'en', 'es'])


@application.route('/')
def handle_no_locale():
    fallback_locale = get_locale().strip()
    return redirect("/" + fallback_locale + request.full_path) 

{% endhighlight %}
Only one problem is left now and this is handling annoying `favicon.ico` requests. Now if we run following setup, favicon.ico will be prefixed as language code and raise exceptions in the database layer, as there is no language code by the name of `favicon.ico`, so to handle this we made a special route to exclusively handle it, above the route the root route to give it more priority

{% highlight python %}
# Handle Annoying Favicon.ico
# TODO - Delegate static asset handling to Proxy Server
# http://flask.pocoo.org/docs/0.12/patterns/favicon/
@application.route('/favicon.ico')
def handle_fav():
    return send_from_directory(
        os.path.join(application.root_path, 'web/static/'),
        'favicon.ico')
{% endhighlight %}


## Goals for Next Week
GSOC final evaluation starts from [21 August to 29 August](https://summerofcode.withgoogle.com/how-it-works/#timeline), and since this a final evaulation which means I need to get everything done from documentation, testing to UI. This week and upcoming week will be busy. I hope to achieve expectations of my mentors and make aslo-v3 a succcesful project.
