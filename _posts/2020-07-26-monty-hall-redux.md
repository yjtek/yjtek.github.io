---
layout: post
title: Monty Hall Redux
categories: []
tags: []
description: An interesting application of the Monty Hall problem
published: true
fullview: false
comments: true
author: Yong Jian
---

<!---
```R
library(tidyverse)
library(gridExtra)
library(kableExtra)
options(kableExtra.html.bsTable = T)
```  
-->

*Adapted from [Drew Dickson](https://www.albertbridgecapital.com/post/heads-i-win-2).*

### Introduction

I took a bit of a hiatus from this blogging thing due to the on-going prep work for my Tensorflow certification. Came across this pseudo Monty Hall problem, and the results are so delightfully unintuitive that I thought I should do a quick post to celebrate this.

### Problem set up
1. Imagine if you're told that a couple has two children, and one of them is a girl. What is the probability that the other child is also a girl?

2. As an extension, what if you're told specifically that the *older* child is a girl? Does this change your answer?

### Discussion
I think it's ridiculously intuitive to simply say 50%, similar to the [Monty Hall Problem](https://en.wikipedia.org/wiki/Monty_Hall_problem). This answer comes from our intuition that children's genders are essentially a coin flip (clearly not a biologist). Having a son doesn't, *shouldn't* increase your odds of subsequently having either a son or a daughter, right?

As it turns out, it really depends on how you frame this problem, which why this is a marvellous lesson on why we can't trust our statistical intuition.

Let's address the first problem by drawing out the possible space of solutions. Given that a couple has 2 children, there are 4 possible gender combinations:

<table class="table table-striped" style="width: auto !important; ">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> Child 1 </th>
   <th style="text-align:right;"> Child 2 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:right;"> Boy </td>
   <td style="text-align:right;"> Boy </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2 </td>
   <td style="text-align:right;"> Boy </td>
   <td style="text-align:right;"> Girl </td>
  </tr>
    <tr>
   <td style="text-align:left;"> 3 </td>
   <td style="text-align:right;"> Girl </td>
   <td style="text-align:right;"> Boy </td>
  </tr>
    <tr>
   <td style="text-align:left;"> 4 </td>
   <td style="text-align:right;"> Girl </td>
   <td style="text-align:right;"> Girl </td>
  </tr>
</tbody>
</table>

Having been told that one of the two children is a girl, that leaves only possibilities 2, 3, and 4. $$\frac{1}{3}$$ of these cases will have a girl as the unknown child, and $$\frac{2}{3}$$ will have a boy. So the answer to our first question is not 50% as we first intuited, but 33%! Let's confirm this via simulation:


```R
child1 <- sample(c('M','F'), 100000, replace = T)
child2 <- sample(c('M','F'), 100000, replace = T)
families <- data.frame(child1, child2)
families %>%
  filter(child1 == 'F' | child2 == 'F') %>%
  mutate(bothGirls = if_else(child1 == 'F' & child2 == 'F', 1, 0)) %>%
  summarise(mean(bothGirls))
```

<table class="table table-striped" style="width: auto !important; ">
<thead>
	<tr><th scope=col>Proportion with both girls</th></tr>
</thead>
<tbody>
	<tr><td>0.3331425</td></tr>
</tbody>
</table>


As expected, conditional on the information that one of the two children is a girl, the remaining child is more likely to be a boy!

What if we knew the birth order of the girl? Suppose now we know that (i) at least 1 child is a girl, and (ii) the first child is a girl. By the logic above, we are now restricted only to cases 3 and 4, and we should now see that the probability of the second child being a girl has increased to 50%.

```R
families %>%
  filter(child1 == 'F') %>%
  mutate(bothGirls = if_else(child2 == 'F', 1, 0)) %>%
  summarise(mean(bothGirls))
```


<table class="table table-striped" style="width: auto !important; ">
<thead>
	<tr><th scope=col>Proportion with both girls</th></tr>
</thead>
<tbody>
	<tr><td>0.499521</td></tr>
</tbody>
</table>


### Conclusion
Notice how the difference between problems 1 and 2 is minimal. The only additional information you get is that it is the first child that is female. Even though this provides us useful information that affects our probability space, it is counter-intuitive that this additional information is even related to the probability of the second child's gender, and is hence discarded!

The takeaway here is that our statistical intuition is a tricky beast to control, so we need to be quite careful about how we draw conclusions in the course of work. A simple case where you can exploit this intuitive failing is by a simple coin flip bet. Flip 2 coins, reveal that one of them comes up heads, and take bets on the probability that the other comes up heads as well. At the "fair" odds of 2:1, you'll be netting a pretty solid profit margin. (:
