---
title: "retrospective"
layout: archive
permalink: /retrospective
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.retrospective %}
{% for post in posts %} {% include archive-categories.html type=page.entries_layout %} {% endfor %}