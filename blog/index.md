---
layout: default
title: "Blog"
post-header: true
header-img: img/aris-attack.jpg
header-class: aris-attack
---

<ul class="catalogue">
{% assign sorted = site.pages | sort: 'order' | reverse %}
{% for page in sorted %}
  {% if page.hidden == true %}{% continue %}{% endif %}
  {% if page.blog == true %}
    {% include post-list.html %}
  {% endif %}
{% endfor %}
</ul>