---
title: "project1"
layout: archive
permalink: /project1
---


{% assign posts = site.categories.project1 %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}