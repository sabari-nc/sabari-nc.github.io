---
layout: archive
title: "Blog"
permalink: /blog/
author_profile: true
---

Welcome to my blog!  
Here, I share field experiences, reflections and insights from my work on landslides, geophysical monitoring and environmental science.

Click on a post preview below to read more.

{% include base_path %}

<div style="display: flex; flex-wrap: wrap; gap: 2rem; justify-content: center;">
{% for post in site.posts %}
  <div class="archive__item" style="flex: 0 1 300px; box-sizing: border-box;">
    <article class="archive__item" itemscope itemtype="https://schema.org/CreativeWork">
      <div class="archive__item-teaser">
        {% if post.image %}
          <a href="{{ post.url | relative_url }}"><img src="{{ post.image | relative_url }}" alt="Preview image for {{ post.title }}" style="width: 250px; height: auto; border-radius: 4px;"></a>
        {% endif %}
      </div>
      <h2 class="archive__item-title" itemprop="headline">
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      </h2>
      <p class="archive__item-excerpt" itemprop="description">{{ post.excerpt }}</p>
    </article>
  </div>
{% endfor %}
</div>