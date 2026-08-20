---
icon: fas fa-flask
order: 1
title: Challenge Authoring
---

This section collects DFIR challenges I designed for CTFs. These posts describe the challenge author's perspective: the investigation story, the artifacts and techniques intentionally included, and the intended solve path.

They are separate from my player write-ups, where I document challenges created by someone else.

## EYCC CTF

The EYCC CTF author write-ups will appear here as they are published.

{% assign authored_posts = site.posts | where_exp: "post", "post.categories contains 'challenge-authoring'" %}
{% if authored_posts.size > 0 %}
{% for post in authored_posts %}
- [{{ post.title }}]({{ post.url | relative_url }}){% if post.description %} — {{ post.description }}{% endif %}
{% endfor %}
{% else %}
The two EYCC DFIR challenge write-ups are coming soon.
{% endif %}
