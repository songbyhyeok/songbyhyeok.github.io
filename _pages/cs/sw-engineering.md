---
title: "software engineering"
layout: archive
permalink: /software-engineering
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.software-engineering %}
{% for post in posts %} {% include archive-categories.html type=page.entries_layout %} {% endfor %}