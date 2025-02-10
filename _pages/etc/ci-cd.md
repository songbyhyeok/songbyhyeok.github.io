---
title: "ci/cd"
layout: archive
permalink: /ci-cd
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.ci-cd %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}