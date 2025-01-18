---
layout: home
---

## Latest Posts

{% for post in site.posts %}
  <article class="post">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    <div class="post-meta">{{ post.date | date: "%B %-d, %Y" }}</div>
    <div class="post-excerpt">
      {{ post.excerpt }}
    </div>
  </article>
{% endfor %}
