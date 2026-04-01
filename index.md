---
layout: default
title: "Prasanna's Blog"
---

<img src="assets/img/ProfilePicBeach.jpg"
     alt="Prasanna Maddila" 
     style="float: right; width: 200px; margin-left: 20px">

Hello ! Welcome to my blog. This is for all things code and math that I find interesting - interesting enough that I take out the time to write a little bit about it, which helps me remember it.

# [News](./news.md)

There's also a list of personal news, but this list is updated asynchronously. Peruse at your own risk, fellow traveller !

# [Posts](./posts.md)

Here's a quick overview of my latest posts. Click on the header to see the whole list.

<table class="news-table">
  <tbody>
    {% for post in site.posts limit:4 %}
      <tr>
        <td style="border: none; padding: 8px 0; text-align: center; color: #888; font-size: 0.9em;">
          {{ post.date | date: "%b %d, %Y" }}
        </td>
        <td style="border: none; padding: 8px 0;">
          <a href="{{ post.url | prepend: site.url }}">{{ post.title }}</a>
        </td>
      </tr>
    {% endfor %}
  </tbody>
</table>

## Contributing and Issues

Feel free to open an issue to a particular page if you find mistakes; I'll be happy to respond :)
