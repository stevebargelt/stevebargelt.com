---
author: Steve Bargelt
categories:
- Health
date: 2016-09-01T00:00:00Z
featured_image: /assets/CHO_CVD.jpg
subtitle: It's not what you'd expect
tags:
- health
- cholesterol
- diet
- medicine
- keto
title: Cholesterol's relationship to cardiovascular disease
url: /2016/09/01/cholesterol-cardiovascular-disease/
---

### Cholesterol's relationship to cardiovascular disease (or lack thereof)
A few months ago I watched a presentation by Zoë Harcombe and in it she referenced her data analysis of World Health Organization data on cholesterol and cardiovascular disease. She mentioned that the relationship between cholesterol numbers and cardiovascular related deaths is the opposite of what you'd expect. I found her original post on the subject: [Zoë Harcombe's cholesterol and heart disease relationship.](http://www.zoeharcombe.com/2010/11/cholesterol-heart-disease-there-is-a-relationship-but-its-not-what-you-think/) 

Here is a snipet of her results: 
![cholesterol vs. cvd deaths chart](/assets/cholesterol001_small.png){: .img-blog }

I was fascinated, but being the data nerd that I am, I was pretty surprised that she did not publish how she came to this conclusion. We can't see behind the scenes. We have no idea how she manipulated the data. In other words, the results were not reproducible. So I wanted to reproduce this result and share all of the data **and code** so that others can critique my methods and results.  Here is my repo with all code and data acquisition code so others can investigate my work: [Github](https://github.com/stevebargelt/WHO-Data).

I headed over to the WHO site and searched. And Searched. AND searched. I finally found what I was looking for. The starting point for all of the WHO data is the [World Health Organization (WHO) Global Health Observatory (GHO)](http://www.who.int/gho/en/). From there I finally stumbled across [cholesterol data](http://apps.who.int/gho/data/node.main.A883?lang=en). 

Finding information on deaths caused by cardiovascular disease wasn't quite as easy. The WHO/GHO splits CVD into ischaemic events (heart attacks) and cerebrovascular events (strokes), so in my analysis I combined the two. 

Here is a snipet of my results:
![cholesterol vs. cvd deaths chart](/assets/cholesterol002_small.png){: .img-blog }

Here is a link to [my full analysis](/WHO_CHO_CVD.html) on this website. I've also placed [my analysis on RPubs](http://rpubs.com/stevebargelt/who-cho-cvd) for public comment.

## Conclusion
It seems from the data that there is no correlation between high cholesterol and cardiovascular disease deaths. As a matter of fact, the opposite seems to be true; countries with higher cholesterol levels have lower CVD death rates than countries with lower cholesterol levels. 

Next steps: I want to compare cholesterol levels with all causes of death, and then compare cholesterol levels with Type II diabetes (and maybe mortality?).
