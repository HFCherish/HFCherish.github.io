---
title: rest and hateoas
toc: true
date: 2019-06-04 15:47:32
tags:
	- rest
---

REST(Representational State Transfer)



[why hateoas is useless and what that means for REST](https://medium.com/@andreasreiser94/why-hateoas-is-useless-and-what-that-means-for-rest-a65194471bc8)

[what does hateoas have to do with Application State](https://softwareengineering.stackexchange.com/questions/384704/what-does-hateoas-have-to-do-with-application-state)

# why REST?

Let's look at a data platform story:

1. The data platform went on-line. A lot of customers are consuming data from the platform.
2. The data platform evolved. Some request urls are changed.
3. Based on the change, the platform has three options:
   1. update the current edition and keep backward compability, thus clients don't need to do anything.
   2. release a completely new edition and keep the old edition at the same time. The clients can choose to update to a new edition.
   3. release a completely new edition and remove the old edition. The clients must update synchronously.



I change 



# what's representation?

# what's application state

# what's hateoas

# why hateoas?

# when do we need to choose REST?

# pain points