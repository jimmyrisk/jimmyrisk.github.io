---
layout: archive
title: "Github"
permalink: /github/
author_profile: true
---

Main link: [github.com/jimmyrisk/](https://github.com/jimmyrisk/)

{% include base_path %}

{% for post in site.github reversed %}
  {% include archive-single.html %}
{% endfor %}
