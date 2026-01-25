---
layout: page
title: "Posts"
---

Here's the complete list of posts on this site.

# Math

These posts are mostly on math I find interesting. You can definitely expect some algebra in there, but other curiosities might find their way here.

<ul>
  {% for post in site.categories.math %}
    <li>
      <a href="{{ post.url | prepend: site.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

## Code 

This is code-heavy - think programming, operating systems and anything to do with performance. As usual, I will try to blog about interesting things.

<ul>
  {% for post in site.categories.code %}
    <li>
      <a href="{{ post.url | prepend: site.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
