---
layout: archive
title:  Articles I have written
author_profile: true
excerpt: "An archive of articles sorted by date."
---

{% include group-by-array collection=site.posts field="categories"  %}

{% for category in group_names %}
  {% assign posts = group_items[forloop.index0] %}
   {% if category == 'articles' %}
  {% for post in posts %}
    {% include archive-single.html %}
  {% endfor %}
  {% endif %}
{% endfor %}
