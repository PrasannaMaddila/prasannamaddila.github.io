---
layout: default
title: "Prasanna's Blog"
---

Hello ! Welcome to my blog. This is for all things code and math that I find interesting - interesting enough that I take out the time to write a little bit about it, which helps me remember it.


# [News](./news.md)

There's also a list of personal news, but this list is updated asynchronously. Peruse at your own risk, fellow traveller !


# Posts 

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | prepend: site.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>


### Contributing and Issues

Please feel free to open an issue to a particular page if you find mistakes, and I'll be happy to respond :)
