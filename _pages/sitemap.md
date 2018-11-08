---
layout: archive
title: "Sitemap"
permalink: /sitemap/
author_profile: true
---

{% include base_path %}

<h2>Talks</h2>
{% for post in site.talks %}
  {% include archive-single.html %}
{% endfor %}

<h2>Research works</h2>
{% for post in site.works %}
  {% include archive-single.html %}
{% endfor %}

<h2>Blog Posts</h2>
{% for post in site.posts %}
  {% include archive-single.html %}
{% endfor %}
