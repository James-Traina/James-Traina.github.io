---
layout: default
title: Teaching
permalink: /teaching/
---
# Teaching

{% assign courses = site.courses | sort: "date" | reverse %}
{% for course in courses %}
<article class="course-entry">
  <h2>{{ course.title }}</h2>
  <p class="course-meta">{{ course.meta }}</p>
  <p>{{ course.summary }}</p>
  <div class="paper-links"><a class="button button-small" href="{{ course.link.url }}" target="_blank" rel="noopener noreferrer">{{ course.link.label }}</a></div>
</article>
{% endfor %}
