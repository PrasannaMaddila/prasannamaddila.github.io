---
layout: default
title: "Prasanna's Blog"
---

Hello ! Welcome to the repository where I store my blog. This is for all things code and math that I find interesting. Interesting enough that I take out the time to write a little bit about it, which helps me remember it. 


# News
- (Workshop, [Mannheim](https://www.wim.uni-mannheim.de/doering/conferences/rl-2025/)) Presented a poster and attended the Workshop on Reinforcement Learning at Mannheim, this 7th February, 2025. 
- (Teaching [Deep Learning](https://forgemia.inra.fr/siva-sri-prasanna.maddila/course-deeplearning-mse-m2)) I've had the pleasure to give a short introductory course on Deep Learning to the students of the MSE Master at the University of Toulouse III :) 
- ([JFSMA 2024](https://easychair.org/cfp/jfsma2024))([Paper](https://hal.science/hal-04699116))  I'll be attending the 32nd (-ième?) Journées Francophones sur les Systèmes Multi-Agents at Cargèse, Corsica this year. Find me there between 5-9 November !
- ([EWRL 2024](https://ewrl.wordpress.com/ewrl17-2024/))([Paper](https://openreview.net/forum?id=JSWRnHC93W&noteId=JSWRnHC93W)) Presenting a poster at the 17th European Workshop on Reinforcement Learning, held at Toulouse this year!
- ([WLiG 2024](https://indico.math.cnrs.fr/event/10543/overview))([Paper](https://hal.science/hal-04701220)) Attended the Workshop on Learning in Games held at the IMT, Toulouse, and presented a poster.
- ([ROADEF 2024](https://roadef2024.sciencesconf.org/?lang=fr))([Paper](https://roadef2024.sciencesconf.org/511462)) Presented the Anti-Poaching Game at ROADEF 2024, held in Amiens. 


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
