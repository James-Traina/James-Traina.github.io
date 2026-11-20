---
layout: default
title: Research
permalink: /research/
---
<h1>Research</h1>

{% assign working = site.papers | where: "category", "working" | sort: "date" | reverse %}
{% if working.size > 0 %}
<section class="section">
<h2 class="section-heading">Working Papers</h2>
{% for paper in working %}{% include paper-entry.html paper=paper %}{% endfor %}
</section>
{% endif %}

{% assign published = site.papers | where: "category", "published" | sort: "date" | reverse %}
{% if published.size > 0 %}
<section class="section">
<h2 class="section-heading">Publications</h2>
{% for paper in published %}{% include paper-entry.html paper=paper %}{% endfor %}
</section>
{% endif %}

{% assign other = site.papers | where: "category", "other" | sort: "date" | reverse %}
{% if other.size > 0 %}
<section class="section">
<h2 class="section-heading">Other Work</h2>
{% for paper in other %}{% include paper-entry.html paper=paper %}{% endfor %}
</section>
{% endif %}
