---
layout: page
permalink: /publications/
title: publications
description:
years_conf:       [2022, 2021, 2019, 2018, 2011]
years_journal:    [2022]
years_workshop:   [2021]
years_tr:         [2021, 2019]
years_patent:     [2021, 2020]
nav: true
---

<div class="publications">
<h3 class="pubtype">Conference</h3>
{% for y in page.years_conf %}
   <h3 class="year">{{y}}</h3> 
   {% bibliography -f conf -q @*[year={{y}}]*%}   
{% endfor %}

<h3 class="pubtype">Journal</h3> 
{% for y in page.years_journal %}
   <h3 class="year">{{y}}</h3> 
  {% bibliography -f journal -q @*[year={{y}}]*%} 
{% endfor %}

<h3 class="pubtype">Workshop</h3> 
{% for y in page.years_workshop %}
   <h3 class="year">{{y}}</h3> 
  {% bibliography -f workshop -q @*[year={{y}}]*%} 
{% endfor %}

<h3 class="pubtype">Technical Reports</h3> 
{% for y in page.years_tr %}
   <h3 class="year">{{y}}</h3> 
  {% bibliography -f techreport -q @*[year={{y}}]*%} 
{% endfor %}

<h3 class="pubtype">Patents</h3> 
{% for y in page.years_patent %}
   <h3 class="year">{{y}}</h3> 
  {% bibliography -f patent -q @*[year={{y}}]*%} 
{% endfor %}

</div>
