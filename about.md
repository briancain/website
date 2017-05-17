---
layout: minimal
title: "About me"
permalink: /about/index.html
description: "Some description about Brian Cain..."
---

<img itemprop="image" class="img-rounded about_perfil" src="/images/brian-new.png" alt="My profile">

Welcome to my site. Here's some information about myself!

<sub>PS: Thank you to <a href="http://duetarts.tumblr.com/" target="_blank">Maya</a> for my icon!</sub>

## Random Facts

- Grew up around the Kansas City area on the Missouri and Kansas sides.
- Musician since the age of 5
- Enjoys learning about tea and coffee (and drinking it too)
- Helped grow a strength community in Manhattan Kansas and currently contributes to one in Portland Oregon.

## Technical Details

### Experience

- HashiCorp (2017-Present)
  + Software Engineer
- Puppet (2014-2017)
  + Software Engineer
- Argus Labs (2013-2014)
  + Software Developer
- Puppet Labs (2013)
  + Engineering Intern
- National Science Foundation (2012-2014)
  + INSIGHT GK-12 Fellow
- Kansas State Cyber Defense Club
  + President
  + Team Captain of Cyber Defense Competition

### Education

- Computer Information Science at Kansas State University
  + _Bachelors of Computer Science (2008)_
  + _Master of Computer Science (2014)_

### Resume

A link to my full resume can be found [here](https://docs.google.com/document/d/1UJNixM8rrRkE5sKoZhxvCyA9F9Rkgcfqf-PP-iwa9nI/edit?usp=sharing).

## Contact

- Email
  + brianccain [at] gmail [dot] com
- irc
  + server: irc.freenode.net
    - nick: `brian

<h2>My projects</h2>

<div class="aboutme">
  <ul class="recent">
    {% for project in site.projects %}
        <li><a href="{{ project.url}}"><h3 class="project-name" itemprop="name">{{ project.name }}</h3></a></li>
      {% endfor %}
  </ul>
</div>
