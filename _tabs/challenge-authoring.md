---
layout: challenge-authoring
icon: fas fa-flask
order: 1
title: Challenge Authoring
---

<div class="content">
This section collects DFIR challenges I designed for CTFs. These posts describe the challenge author’s perspective: the investigation story, the artifacts and techniques intentionally included, and the intended solve path.

They appear on the home page like the rest of the blog posts and are separate from player write-ups, where I document challenges created by someone else.
</div>

{% assign authored_posts = site.posts | where_exp: "post", "post.categories contains 'challenge-authoring'" %}
{% include lang.html %}

<div id="post-list" class="flex-grow-1 px-xl-1">
  {% for post in authored_posts %}
    {% include challenge-post-card.html post=post %}
  {% endfor %}
</div>
