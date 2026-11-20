---
layout: default
title: Teaching
permalink: /teaching/
---
<h1>Teaching</h1>

{% assign courses = site.courses | sort: "date" | reverse %}
{% for course in courses %}
<article class="course-entry">
  <h2>{{ course.title }}</h2>
  <p class="course-meta">{{ course.meta }}</p>
  <p>{{ course.summary }}</p>
  {% if course.link and course.link.url %}<div class="entry-links"><a href="{{ course.link.url }}" target="_blank" rel="noopener noreferrer">{{ course.link.label }}</a></div>{% endif %}
</article>
{% endfor %}
