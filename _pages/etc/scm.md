---
title: "scm"
layout: archive
permalink: /scm
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.scm %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}