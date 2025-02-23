---
title: "temp"
layout: archive
permalink: /temp
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.temp %}
{% for post in posts %} 
    {% include archive-categories.html type=page.entries_layout %} 
{% endfor %}