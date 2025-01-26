---
title: "retrospective"
layout: archive
permalink: /categories/retrospective
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.retrospective %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}