---
title: Welcome to Olivando's World
layout: splash
permalink: /
hidden: true

header:
  overlay_filter: 0.5
  overlay_image: assets/images/home-splash.jpg
  caption: "Photo credit: [**Unsplash**](https://unsplash.com){:target='_blank'}"

  actions:
    - label: Who am I?
      url: /about-me/

excerpt: A documentation of my adventures in IT/CS/Homelab

intro:
    - excerpt: Unsure of where to start? Select an adventure below

feature_row:
  - title: Adventures in Homelab
    excerpt: A journey into Infrastructure management, Virtualisation, Containerisation, Orchestration and so much more

    url: /homelab/
    btn_label: Read more
    btn_class: btn--primary 

feature_row2:
  - title: Adventures in Programming
    excerpt: An adventure fraught with Puzzling problems, "weekend long" side-quests and many, many bugs

    url: /programming/
    btn_label: Under Construction
    btn_class: btn--primary 
---

{% include feature_row id="intro" type="center" %}

{% include feature_row type="center" %}

{% include feature_row id="feature_row2" type="center" %}
