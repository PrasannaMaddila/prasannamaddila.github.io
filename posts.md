---
layout: page
title: "Posts"
---

Here's the complete list of posts on this site.

# Math

These posts are mostly on math I find interesting. You can definitely expect some algebra in there, but other curiosities might find their way here.

<table style="width: 100%; border: none; border-collapse: collapse;">
  <tbody>
    {% for post in site.categories.math %}
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

## Code 

This is code-heavy - think programming, operating systems and anything to do with performance. As usual, I will try to blog about interesting things.

<table style="width: 100%; border: none; border-collapse: collapse;">
  <tbody>
    {% for post in site.categories.code %}
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
<ul>
