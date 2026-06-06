---
layout: default
title: Research
permalink: /research/
---
# Research

## Working Papers

{% assign working = site.papers | where: "category", "working" | sort: "date" | reverse %}
{% for paper in working %}{% include paper-entry.html paper=paper %}{% endfor %}

## Publications

{% assign published = site.papers | where: "category", "published" | sort: "date" | reverse %}
{% for paper in published %}{% include paper-entry.html paper=paper %}{% endfor %}

## Other Writing

{% assign other = site.papers | where: "category", "other" | sort: "date" | reverse %}
{% for paper in other %}{% include paper-entry.html paper=paper %}{% endfor %}
