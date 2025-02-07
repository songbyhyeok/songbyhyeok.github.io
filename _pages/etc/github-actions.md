---
title: "githubActions"
layout: archive
permalink: /github-actions
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.github-actions %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}