---
author: Spencer Boucher
layout: reveal
theme: black
title: testing reveal slides
transition: slide
---

{% slide %}

## This is a slide with just an h2 tag

- foo
- bar
- baz

{% endslide %}

{% slide %}
{% slide %}
## This is a vertical slide

ooh so vert

{% endslide %}

{% slide %}
## This is the second vertical slide

![this is an image]({{ site.baseurl }}/images/avatar.jpg)

{% endslide %}
{% endslide %}

{% slide data-background="#dddddd" %}
## This slide will have a different background color
{% endslide %}
