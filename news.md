---
layout: page
title: "News"
---

I try to organise personal news by theme so it's easier to walk through. Better organisation suggestions are always welcome (and might even be necessary) !

## Papers and Publications

Papers from my PhD work - this is mostly around Algorithmic Game Theory (Nash Equilibrium Computatation) and MARL for the moment.

### Multi-Adversarial Team Games

Multi-Adversarial Team Games model the team vs. independent adversary dynamic that is central to APE. They also generalise the class of Adversarial Team Games, classically considered in Game Theory. 

- We model this first as a normal form game, where we show that equilibria in this game are poly-time approximable. [Code: MATG](https://forge.inrae.fr/chip-gt/multi-adversarial-team-games) 
- We then consider the Markov extension of this game, and identify the tractability boundary (PPAD-Hardness vs. existence of FPTAS). For the tractable class (additive transition dynamics), we provide an implementation that scales well. [Code: FHAT-MATMG](https://forge.inrae.fr/chip-gt/fhat_matmg)

<div class="remark" text="Code">
All provided code is JAX-native, GPU-compatible and is tested for correctness (PyTest). Do contact me if you have any bugs, bug-fixes, suggestions, improvements, changes you'd like to see, or any new use-cases you want to try this code out on! 
</div>

<table class="news-table">
  <tbody>
    {% for item in site.data.matg %}
      <tr>
        <td>
            <span class="news-date">{{ item.display_date | markdownify }}</span>
            <span class="news-tag">{{ item.tag | markdownify }}</span>
        </td>
        <td class="news-content">
          {{ item.content | markdownify | remove: '<p>' | remove: '</p>' }}
        </td>
      </tr>
    {% endfor %}
  </tbody>
</table>

### Anti-Poaching Environment 

This is the body of work we produced on modelling Anti-Poaching as a partially observable stochastic game. This is the first environment in this space with a theoretical model and a reference implementation.

[Code: APE](https://forge.inrae.fr/chip-gt/antipoaching) Implemented using PettingZoo and comes with some MARL algorithms written in Ray/RLlib.

<table class="news-table">
  <tbody>
    {% for item in site.data.ape %}
      <tr>
        <td>
            <span class="news-date">{{ item.display_date | markdownify }}</span>
            <span class="news-tag">{{ item.tag | markdownify }}</span>
        </td>
        <td class="news-content">
          {{ item.content | markdownify | remove: '<p>' | remove: '</p>' }}
        </td>
      </tr>
    {% endfor %}
  </tbody>
</table>

## Teaching

In addition to my PhD work, I also teach at the University of Toulouse. This is usually a 12-hour primer on Deep Learning for the students of the MSE Master.

<table class="news-table">
  <tbody>
    {% for item in site.data.teaching %}
      <tr>
        <td>
            <span class="news-date">{{ item.display_date | markdownify }}</span>
            <span class="news-tag">{{ item.tag | markdownify }}</span>
        </td>
        <td class="news-content">
          {{ item.content | markdownify | remove: '<p>' | remove: '</p>' }}
        </td>
      </tr>
    {% endfor %}
  </tbody>
</table>
