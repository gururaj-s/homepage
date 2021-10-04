---
layout: page
permalink: /publications/
title: publications
description:
years_conf:       [2021, 2019, 2018, 2011]
years_workshop:   [2021]
years_tr:         [2019]
years_patent:     [2021, 2020]
nav: true
---

<div class="publications">
<h2>Conference</h2>
{% for y in page.years_conf %}
   <h2 class="year">{{y}}</h2> 
   {% bibliography -f conf -q @*[year={{y}}]*%}   
{% endfor %}

<h2>Workshop</h2> 
{% for y in page.years_workshop %}
   <h2 class="year">{{y}}</h2> 
  {% bibliography -f workshop -q @*[year={{y}}]*%} 
{% endfor %}

<h2>Technical Reports</h2> 
{% for y in page.years_tr %}
   <h2 class="year">{{y}}</h2> 
  {% bibliography -f techreport -q @*[year={{y}}]*%} 
{% endfor %}

<h2>Patents</h2> 
{% for y in page.years_patent %}
   <h2 class="year">{{y}}</h2> 
  {% bibliography -f patent -q @*[year={{y}}]*%} 
{% endfor %}

</div>
