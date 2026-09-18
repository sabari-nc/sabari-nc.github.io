---
layout: portfolio
title: "Writing"
permalink: /blog/
excerpt: "Notes on geoscience, the environment, and working with spatial data."
---
<section class="page-heading"><h1>Writing</h1><p>Geoscience, the environment, and working with spatial data.</p></section>
<div class="writing-list">
{% for post in site.posts %}
<article class="writing-card"><time datetime="{{ post.date | date: '%Y-%m-%d' }}">{{ post.date | date: '%d %B %Y' }}</time><h2><a href="{{ post.url | relative_url }}">{{ post.title | escape }}</a></h2><a href="{{ post.url | relative_url }}">Read article</a></article>
{% endfor %}
</div>
