---
layout: default
title: Research
permalink: /research/
---
# Research

{% assign working = site.papers | where: "category", "working" | sort: "date" | reverse %}
{% if working.size > 0 %}
## Working Papers

{% for paper in working %}{% include paper-entry.html paper=paper %}{% endfor %}
{% endif %}

{% assign published = site.papers | where: "category", "published" | sort: "date" | reverse %}
{% if published.size > 0 %}
## Publications

{% for paper in published %}{% include paper-entry.html paper=paper %}{% endfor %}
{% endif %}

{% assign other = site.papers | where: "category", "other" | sort: "date" | reverse %}
{% if other.size > 0 %}
## Other Work

{% for paper in other %}{% include paper-entry.html paper=paper %}{% endfor %}
{% endif %}
