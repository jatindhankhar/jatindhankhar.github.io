---
title: GSOC Sugar Labs - Week 16
categories: blog
excerpt: 'GSOC Suar Labs - Week 16'
tags: [programming,sugar labs,gsoc]
comments: true
description: 'GSOC Suar Labs - Week 16'

---
This is the sixteenth post in the series of my weekly GSOC Sugar Labs, where I summarize my week of working with [Sugar Labs](https://www.sugarlabs.org) under GSOC.

## Weekly Journal 
This week Samuel and I worked on adding translations for Spanish and Hindi version along with migration of old aslo-1 data. Also, aslo-v3 has new home, website is hosted here [https://aslo3-devel.sugarlabs.org/en/](https://aslo3-devel.sugarlabs.org/en/) and source code shifted to [Sugarlabs Github Repo](https://github.com/sugarlabs/aslo-v3).

#### Adding translations
Samuel added the Spanish `es` translation ([https://aslo3-devel.sugarlabs.org/es/](https://aslo3-devel.sugarlabs.org/es/)) and I worked on the Hindi `hi` translation ([https://aslo3-devel.sugarlabs.org/hi/](https://aslo3-devel.sugarlabs.org/hi/)). For handling translations we are using `flask-pybabel`.  User locale is set by either retrieving session code or predicting from browser headers.
{% highlight python %}

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

def get_app_locale():
    if 'lang_code' in session:
        return session['lang_code']
    else:
        return get_language()

  # init Babel
    babel = Babel(app)
    babel.localeselector(get_app_locale)
{% endhighlight %}

Then each translated `.po` file is added to the appropriate destination and translated by the babel.


#### Migration Script
This was the most difficult and fun of all. We had to migrate old aslo-1 data to new aslo-v3 and adapt it to the new schema as well. I encountered many edge cases there (like empty icon data, no translation, no license). Migration script started out as simple but ended up being a messy script :sweat_smile:. To access mysql data (which was imported out of a dump) I used `sqlsoup` which is built upon `sqlAlchemy` (sqlAlchemy is also good but sqlsoup automatically infers schema and provides a high level control over database). To insert the actual data we re-used the `Activity service`. Migration script is by no means complete but we will get there. To aid up the analysis of addons (activity), script logs the interesting facts to disk. 

Here is the [Gist to the script](https://gist.github.com/jatindhankhar/91ea2d79674e6874f68b87e917e8353a)
 and here is the code 

 {% highlight python %}
 # Requires Python 3.2 and onwards

import sqlsoup
import itertools
import os


from aslo.service import activity as activity_service

db = sqlsoup.SQLSoup("mysql://{}:{}@{}/{}".format("root",
                                                  "YourPassword", "localhost", "activities"))


# Setup log directories
# Python 3.2 onwards
os.makedirs("faulty_addons/", exist_ok=True)
os.makedirs("good_addons/", exist_ok=True)


import mongoengine as me
me.connect('aslo')


def make_user_hash(user_info):
    user_hash = dict.fromkeys(['name', 'page', 'email', 'avatar'], None)
    user_hash["name"] = user_info.firstname + ' ' + user_info.lastname
    user_hash["page"] = (user_info.homepage or None)
    user_hash["email"] = user_info.email
    return user_hash


def modify_locale_underscore(translation_entry):
    # Both tuples and dict keys are immutable :(
    new_locale, localized_string = translation_entry[0].replace(
        "-", "_"), translation_entry[1]
    return (new_locale, localized_string)


def convert_translations(translation_id, without_locale=False, singular=False):
   # Handle Null/None translation ids
    if translation_id is None:
        return None

    if without_locale:
        if singular:
            translation = db.translations.with_entities(db.translations.localized_string).filter(
                db.translations.id == translation_id).first()
        else:
            translation = db.translations.with_entities(db.translations.localized_string).filter(
                db.translations.id == translation_id).all()
        return translation
    else:
        translation = db.translations.with_entities(db.translations.locale, db.translations.localized_string).filter(
            db.translations.id == translation_id).all()
        translation = list(map(modify_locale_underscore, translation))
        return dict(translation)


def find_developers(addon_id):
    user_ids = db.addons_users.with_entities(db.addons_users.user_id).filter(
        db.addons_users.addon_id == addon_id).all()
    user_lists = []
    for user in user_ids:
        user_info = db.users.filter(db.users.id == user[0]).one()
        user_lists.append(make_user_hash(user_info))
    return user_lists


def get_bundle_details(version_id):
    result = db.files.with_entities(db.files.filename, db.files.created).filter(
        db.files.version_id == version_id)
    if result.count() > 0:
        return result.one()
    else:
        return None, None

# App version id is different from version id


def get_sugar_version(app_version_id):
    return db.appversions.filter(db.appversions.id == app_version_id).one().version


def get_license(license_id):

    FALLBACK_LICENSE = None
    if license_id is None:
        return FALLBACK_LICENSE

    text_id = db.licenses.filter(db.licenses.id == license_id).one().text
    if text_id is None:
        return FALLBACK_LICENSE
    # Strip license with long texts and return header only
    license_text = convert_translations(text_id, without_locale=True, singular=True)[
        0].strip().split("\r")[0].strip()
    if not license_text:
        return FALLBACK_LICENSE
    else:
        return license_text


def flatten_singly_tuple_list(target_list):
    def single_out(el): return el[0]
    return list(map(single_out, target_list))


def get_sugar_info(version_id):
    sugar_info = dict.fromkeys(
        ['is_web', 'is_gtk3', 'has_old_toolbars'], False)
    version_info = db.applications_versions.filter(
        db.applications_versions.version_id == version_id)
    if version_info.count() > 0:
        sugar_info['min_sugar_version'] = get_sugar_version(
            version_info.one().min)
    else:
        sugar_info['min_sugar_version'] = 0.0

    return sugar_info


def get_category_text(category_id):
    translation_id = db.categories.filter(
        db.categories.id == category_id).one().name
    return flatten_singly_tuple_list(convert_translations(translation_id, without_locale=True))


def get_addon_categories(addon_id):
    category_ids = db.addons_categories.with_entities(
        db.addons_categories.category_id).filter(db.addons_categories.addon_id == addon_id).all()
    category_ids = flatten_singly_tuple_list(category_ids)
    categories = list(map(get_category_text, category_ids))
    # Flattens a nested list
    return list(itertools.chain(*categories))


def generate_old_download_url(bundle_name, addon_id):
    return "http://activities.sugarlabs.org/activities/{}/{}".format(addon_id, bundle_name)


def get_addon_releases_info(addon):
    addon_id = addon.id
    releases = []

    def isFloat(el):
        try:
            float(el[1])
            return True
        except:
            with open("faulty_addons/{}.log".format(addon_id), 'a') as f:
                print("Skipping version id {} and  activity version {}. Reason: Invalid/non-float version number ".format(
                    el[0], el[1]), file=f)
            return False

    version_info = db.versions.with_entities(db.versions.id,
                                             db.versions.version, db.versions.releasenotes, db.versions.license_id).filter(db.versions.addon_id == addon_id).all()

    # Remove non float versions
    version_info = [el for el in version_info if isFloat(el)]
    # Couldn't sqlachemy way of supplying custom function of order, there is one out there but I did a hack of sorting the result
    # Sort by float value of versions
    # We are sorting to avoid error by activity_service

    version_info = sorted(version_info, key=lambda el: float(el[1]))
    # Convert release notes/
    i18n_name = convert_translations(addon.name)
    i18n_summary = convert_translations(addon.description)
    # Add placeholder in case of no summary
    if i18n_summary is None:
        i18n_summary = {'en': 'No Summary '}

    homepage = convert_translations(
        addon.homepage, without_locale=True, singular=True)
    # Handle empty links
    if homepage is not None:
        homepage = homepage[0]
    # Else fallback to support url and then to emptyd
    else:
        supporturl = convert_translations(
            addon.supporturl, without_locale=True, singular=True)
        if supporturl is not None:
            homepage = supporturl[0]
        else:
            homepage = " "

    developers = find_developers(addon_id)
    icon_bin = addon.icondata

    bundle_id = addon.guid

    # First look for license in the suggested amount
    license = addon.suggested_amount

    for version_id, version, releasenote, license_id in version_info:
        release_info = {}
        release_info['release'] = {}
        release_info['screenshots'] = {}
        releasenote = convert_translations(
            releasenote, without_locale=True, singular=True)
        release_info['sugar'] = get_sugar_info(version_id)
        release_info['activity_version'] = version
        bundle_name, release_info['release']['time'] = get_bundle_details(
            version_id)
        if bundle_name is None:
            with open("faulty_addons/{}.log".format(addon_id), 'a') as f:
                print("Skipping version id {} and  activity version {}. Reason: No bundle file found ".format(
                    version_id, version), file=f)
            continue

        # Priortize suggested amount, if None then only get license from license id
        if license is None:
            release_info['license'] = get_license(license_id)
        else:
            release_info['license'] = license

        release_info['download_url'] = generate_old_download_url(
            bundle_name=bundle_name, addon_id=addon_id)

        release_info['i18n_name'] = i18n_name
        release_info['i18n_summary'] = i18n_summary
        release_info['developers'] = developers
        release_info['repository'] = homepage
        release_info['icon_bin'] = icon_bin
        release_info['bundle_id'] = bundle_id
        if releasenote is not None:
            release_info['release']['notes'] = releasenote[0]
        else:
            release_info['release']['notes'] = " "

        if icon_bin is None:
            with open("faulty_addons/{}.log".format(addon_id), 'a') as f:
                print("Skipping version id {} and  activity version {}. Reason: No/Invalid icon data found ".format(
                    version_id, version), file=f)
            continue

        # Skip Activities with unknown, undefined, and no licenses
        if release_info['license'] is not None:
            releases.append(release_info)
            with open("good_addons/{}.log".format(addon_id), 'a') as f:
                print("Added to Database .  version id {} and  activity version {}".format(
                    version_id, version), file=f)
            activity_service.insert_activity(release_info)
        else:
            with open("faulty_addons/{}.log".format(addon_id), 'a') as f:
                print("Skipping version id {} and  activity version {}. Reason: Unknown license ".format(
                    version_id, version), file=f)
    return releases


if __name__ == "__main__":
    non_zero = 0
    zero = 0
    for addon in db.addons.all():
        print("Addon id {}".format(addon.id))
        result = get_addon_releases_info(addon)
        print(find_developers(addon.id))
        if len(result) > 0:
            non_zero = non_zero + 1
            print("Non zero release")
        else:
            zero = zero + 1
            print("Zero release")

    print("Non-zero activities : {}".format(non_zero))
    print("Zero activities : {} ".format(zero))
 {% endhighlight %}
 
As per script logs.

<blockquote> We got 51 activities with 735 releases. 
We rejected releases due to one of the following reasons.

<ul>
 <li> Missing icon data </li>
<li> Unknown license </li>
<li> Non float version numbers </li>
<li> Missing bundles </li>
</ul>
</blockquote>
## Goals for Next Week

This will be the last week of GSOC (GSOC formally ends on 29 September, 2017) but aslo-v3 development will continue :smile:. 
I hope to finish majority of the work this week and make aslo-v3 a viable successor to aslo-1.
