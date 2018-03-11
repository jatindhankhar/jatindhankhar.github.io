---
layout: archive
---

<h3 class="archive__subtitle">{{ site.data.ui-text[site.locale].recent_posts | default: "Recent Posts" }}</h3>

 <div id="index">
    <h1>{{ page.title }}</h1>
    {% for post in site.posts %}  
    {% unless post.next %}
      <h3>{{ post.date | date: '%Y' }}</h3>
      <hr style="border-top: 4px solid #f2f3f3;"/>
      {% else %}
        {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %}
        {% capture nyear %}{{ post.next.date | date: '%Y' }}{% endcapture %}
        {% if year != nyear %}
          <h3>{{ post.date | date: '%Y' }}</h3>
          <hr style="border-top: 4px solid #f2f3f3;"/>
        {% endif %}
      {% endunless %}
      <article>
        {% if post.link %}
          <h2 class="link-post"><a href="{{ site.url }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a> <a href="{{ post.link }}" target="_blank" title="{{ post.title }}"><i class="fa fa-link"></i></a></h2>
        {% else %}
          <h2><a href="{{ site.url }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a></h2>
          <p>{{ post.excerpt | strip_html | truncate: 160 }}</p>
        {% endif %}
      </article>
    {% endfor %}
  </div><!-- /#index -->