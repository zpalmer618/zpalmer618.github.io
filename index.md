---
layout: default
title: Home
---

# Welcome!

To learn more about me click on the **About** button! To see my recent blog posts, check out the **Blogs** tab. Thanks for visiting!

{% for post in site.posts %}
  - {{ post.title }}
{% endfor %}