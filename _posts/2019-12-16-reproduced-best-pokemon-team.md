---
layout: post
title: Pokemon Team Optimisation
categories: []
tags: []
description: Does the selection of pokemon teams reach a Nash Equilibrium upon optimal information?
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

*Adapted from [Emily Robinson](https://hookedondata.org/pokemon-type-combinations/). Data from [robinsones](https://github.com/robinsones/pokemon-chart/blob/master/chart.csv).*

### Introduction

Came across this fairly interesting post recently. Given that there are 18 types of pokemon, some of which are super-effective/not very effective against each other, how do we pick a pokemon team combination that maximises effectiveness against the bulk of other teams? We take in a base dataset showing the effects from each 18*18 attack/defence pair. 0.5 indicates a not very effective attack, 1 indicates a normal attack, and 2 indicates a super effective attack. 



```R
chart <- read.csv('~/Desktop/yjtek.github.io/data/2019-12-16-reproduced-best-pokemon-team/chart.csv')
chart
```


<table class="table table-striped" style="width: auto !important; ">
<thead>
	<tr><th scope=col>Attacking</th><th scope=col>Normal</th><th scope=col>Fire</th><th scope=col>Water</th><th scope=col>Electric</th><th scope=col>Grass</th><th scope=col>Ice</th><th scope=col>Fighting</th><th scope=col>Poison</th><th scope=col>Ground</th><th scope=col>Flying</th><th scope=col>Psychic</th><th scope=col>Bug</th><th scope=col>Rock</th><th scope=col>Ghost</th><th scope=col>Dragon</th><th scope=col>Dark</th><th scope=col>Steel</th><th scope=col>Fairy</th></tr>
</thead>
<tbody>
	<tr><td>Normal  </td><td>1</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>0.5</td><td>0.0</td><td>1.0</td><td>1.0</td><td>0.5</td><td>1.0</td></tr>
	<tr><td>Fire    </td><td>1</td><td>0.5</td><td>0.5</td><td>1.0</td><td>2.0</td><td>2.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>0.5</td><td>1.0</td><td>0.5</td><td>1.0</td><td>2.0</td><td>1.0</td></tr>
	<tr><td>Water   </td><td>1</td><td>2.0</td><td>0.5</td><td>1.0</td><td>0.5</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>0.5</td><td>1.0</td><td>1.0</td><td>1.0</td></tr>
	<tr><td>Electric</td><td>1</td><td>1.0</td><td>2.0</td><td>0.5</td><td>0.5</td><td>1.0</td><td>1.0</td><td>1.0</td><td>0.0</td><td>2.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>0.5</td><td>1.0</td><td>1.0</td><td>1.0</td></tr>
	<tr><td>Grass   </td><td>1</td><td>0.5</td><td>2.0</td><td>1.0</td><td>0.5</td><td>1.0</td><td>1.0</td><td>0.5</td><td>2.0</td><td>0.5</td><td>1.0</td><td>0.5</td><td>2.0</td><td>1.0</td><td>0.5</td><td>1.0</td><td>0.5</td><td>1.0</td></tr>
	<tr><td>Ice     </td><td>1</td><td>0.5</td><td>0.5</td><td>1.0</td><td>2.0</td><td>0.5</td><td>1.0</td><td>1.0</td><td>2.0</td><td>2.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>0.5</td><td>1.0</td></tr>
	<tr><td>Fighting</td><td>2</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>0.5</td><td>1.0</td><td>0.5</td><td>0.5</td><td>0.5</td><td>2.0</td><td>0.0</td><td>1.0</td><td>2.0</td><td>2.0</td><td>0.5</td></tr>
	<tr><td>Poison  </td><td>1</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>1.0</td><td>0.5</td><td>0.5</td><td>1.0</td><td>1.0</td><td>1.0</td><td>0.5</td><td>0.5</td><td>1.0</td><td>1.0</td><td>0.0</td><td>2.0</td></tr>
	<tr><td>Ground  </td><td>1</td><td>2.0</td><td>1.0</td><td>2.0</td><td>0.5</td><td>1.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>0.0</td><td>1.0</td><td>0.5</td><td>2.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>1.0</td></tr>
	<tr><td>Flying  </td><td>1</td><td>1.0</td><td>1.0</td><td>0.5</td><td>2.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>0.5</td><td>1.0</td><td>1.0</td><td>1.0</td><td>0.5</td><td>1.0</td></tr>
	<tr><td>Psychic </td><td>1</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>2.0</td><td>1.0</td><td>1.0</td><td>0.5</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>0.0</td><td>0.5</td><td>1.0</td></tr>
	<tr><td>Bug     </td><td>1</td><td>0.5</td><td>1.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>0.5</td><td>0.5</td><td>1.0</td><td>0.5</td><td>2.0</td><td>1.0</td><td>1.0</td><td>0.5</td><td>1.0</td><td>2.0</td><td>0.5</td><td>0.5</td></tr>
	<tr><td>Rock    </td><td>1</td><td>2.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>0.5</td><td>1.0</td><td>0.5</td><td>2.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>0.5</td><td>1.0</td></tr>
	<tr><td>Ghost   </td><td>0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>0.5</td><td>1.0</td><td>1.0</td></tr>
	<tr><td>Dragon  </td><td>1</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>0.5</td><td>0.0</td></tr>
	<tr><td>Dark    </td><td>1</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>0.5</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>0.5</td><td>1.0</td><td>0.5</td></tr>
	<tr><td>Steel   </td><td>1</td><td>0.5</td><td>0.5</td><td>0.5</td><td>1.0</td><td>2.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>0.5</td><td>2.0</td></tr>
	<tr><td>Fairy   </td><td>1</td><td>0.5</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>0.5</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>1.0</td><td>2.0</td><td>2.0</td><td>0.5</td><td>1.0</td></tr>
</tbody>
</table>



### Basic analysis

We can already draw some basic aggregate conclusions about the most useful types by maximising the attack dealt and minimising the attack received. Remember that the baseline for both scores is 18 (i.e. you deal normal damage to all other types, and take normal damage from all other types). 


```R
mostUsefulAttack <- data.frame(type = chart$$Attacking, `Attack Dealt` = rowSums(chart[,2:ncol(chart)]))
mostUsefulDefence <- data.frame(type = colnames(chart)[2:ncol(chart)], `Attack Received` = colSums(chart[, 2:ncol(chart)]))
mostUseful <- mostUsefulAttack %>% left_join(mostUsefulDefence, by = 'type') %>% `colnames<-`(c('Type', 'Attack Dealt', 'Attack Received'))
mostUseful
```


<table class="table table-striped" style="width: auto !important; ">
<thead>
	<tr><th scope=col>Type</th><th scope=col>Attack Dealt</th><th scope=col>Attack Received</th></tr>
</thead>
<tbody>
	<tr><td>Normal  </td><td>16.0</td><td>18.0</td></tr>
	<tr><td>Fire    </td><td>20.0</td><td>18.0</td></tr>
	<tr><td>Water   </td><td>19.5</td><td>18.0</td></tr>
	<tr><td>Electric</td><td>17.5</td><td>17.5</td></tr>
	<tr><td>Grass   </td><td>17.5</td><td>21.0</td></tr>
	<tr><td>Ice     </td><td>20.0</td><td>21.5</td></tr>
	<tr><td>Fighting</td><td>19.5</td><td>19.5</td></tr>
	<tr><td>Poison  </td><td>17.0</td><td>17.5</td></tr>
	<tr><td>Ground  </td><td>21.0</td><td>19.0</td></tr>
	<tr><td>Flying  </td><td>19.5</td><td>18.5</td></tr>
	<tr><td>Psychic </td><td>18.0</td><td>20.0</td></tr>
	<tr><td>Bug     </td><td>17.5</td><td>19.5</td></tr>
	<tr><td>Rock    </td><td>20.5</td><td>21.0</td></tr>
	<tr><td>Ghost   </td><td>18.5</td><td>17.0</td></tr>
	<tr><td>Dragon  </td><td>17.5</td><td>19.0</td></tr>
	<tr><td>Dark    </td><td>18.5</td><td>19.0</td></tr>
	<tr><td>Steel   </td><td>19.0</td><td>15.0</td></tr>
	<tr><td>Fairy   </td><td>19.5</td><td>17.5</td></tr>
</tbody>
</table>



A cursory analysis will already tell us which types are most useful. If we are willing to ignore the distribution of the scores, we can simply find all types where attack dealt exceeds the baseline, and the attack received is below the baseline.


```R
mostUseful %>%
  filter(`Attack Dealt` >= 18 & `Attack Received` <= 18)
```


<table class="table table-striped" style="width: auto !important; ">
<thead>
	<tr><th scope=col>Type</th><th scope=col>Attack Dealt</th><th scope=col>Attack Received</th></tr>
</thead>
<tbody>
	<tr><td>Fire </td><td>20.0</td><td>18.0</td></tr>
	<tr><td>Water</td><td>19.5</td><td>18.0</td></tr>
	<tr><td>Ghost</td><td>18.5</td><td>17.0</td></tr>
	<tr><td>Steel</td><td>19.0</td><td>15.0</td></tr>
	<tr><td>Fairy</td><td>19.5</td><td>17.5</td></tr>
</tbody>
</table>



### Maximising super-effectiveness

As marvellous as that sounds, we clearly don't fight pokemon in some weird aggregated group fight. Pokemon fights are a 1v1 affair, and it follows that the analysis should be conducted at the type pair level. We sharpen the granularity of the analysis to see which types provide the most "super effective" attacks.


```R
chartLong <- chart %>%
  pivot_longer(-Attacking, names_to = 'Defending', values_to = 'Attack Effectiveness') %>%
  mutate(`Attack Effectiveness` = if_else(`Attack Effectiveness` == 2, 1, 0)) %>%
  group_by(Attacking) %>%
  summarise(`Count Attack Super Effective` = sum(`Attack Effectiveness`)) %>%
  arrange(desc(`Count Attack Super Effective`))
chartLong
```

<table class="table table-striped" style="width: auto !important; ">
<thead>
	<tr><th scope=col>Attacking</th><th scope=col>Count Attack Super Effective</th></tr>
</thead>
<tbody>
	<tr><td>Fighting</td><td>5</td></tr>
	<tr><td>Ground  </td><td>5</td></tr>
	<tr><td>Fire    </td><td>4</td></tr>
	<tr><td>Ice     </td><td>4</td></tr>
	<tr><td>Rock    </td><td>4</td></tr>
	<tr><td>Bug     </td><td>3</td></tr>
	<tr><td>Fairy   </td><td>3</td></tr>
	<tr><td>Flying  </td><td>3</td></tr>
	<tr><td>Grass   </td><td>3</td></tr>
	<tr><td>Steel   </td><td>3</td></tr>
	<tr><td>Water   </td><td>3</td></tr>
	<tr><td>Dark    </td><td>2</td></tr>
	<tr><td>Electric</td><td>2</td></tr>
	<tr><td>Ghost   </td><td>2</td></tr>
	<tr><td>Poison  </td><td>2</td></tr>
	<tr><td>Psychic </td><td>2</td></tr>
	<tr><td>Dragon  </td><td>1</td></tr>
	<tr><td>Normal  </td><td>0</td></tr>
</tbody>
</table>



To form the best combination of 6, and lacking some sort of prior about what that combination should be, we rely on good old fashioned brute forcing. Using `combn()`, we get a dataframe where every column is 1 combination. Using the base dataframe, we change the values such that super effective attacks are reflected as 1s in the matrix, and everything else is reflected as 0.


```R
combinations <- combn(18,6) #get all 6 value combinations of seq(1, 18). Basically this is every possible combination of pokemon teams
m <- as.matrix(chart[, -1]) #get matrix of attack details without the column of types
rownames(m) <- chart$$Attacking
super_effective_m <- (m == 2) * 1L
super_effective_m #if super-effective, return 1. Otherwise return 0
```


<table class="table table-striped" style="width: auto !important; ">
<thead>
	<tr><th></th><th scope=col>Normal</th><th scope=col>Fire</th><th scope=col>Water</th><th scope=col>Electric</th><th scope=col>Grass</th><th scope=col>Ice</th><th scope=col>Fighting</th><th scope=col>Poison</th><th scope=col>Ground</th><th scope=col>Flying</th><th scope=col>Psychic</th><th scope=col>Bug</th><th scope=col>Rock</th><th scope=col>Ghost</th><th scope=col>Dragon</th><th scope=col>Dark</th><th scope=col>Steel</th><th scope=col>Fairy</th></tr>
</thead>
<tbody>
	<tr><th scope=row>Normal</th><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Fire</th><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td></tr>
	<tr><th scope=row>Water</th><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Electric</th><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Grass</th><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Ice</th><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Fighting</th><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td></tr>
	<tr><th scope=row>Poison</th><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td></tr>
	<tr><th scope=row>Ground</th><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td></tr>
	<tr><th scope=row>Flying</th><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Psychic</th><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Bug</th><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Rock</th><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Ghost</th><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Dragon</th><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Dark</th><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Steel</th><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td></tr>
	<tr><th scope=row>Fairy</th><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td></tr>
</tbody>
</table>



Now we have an 18x18 matrix with 1s where attacks are super effective, a 6x18564 matrix of possible combinations of 6 pokemon teams.<sup id="a1">[1](#f1)</sup>. To work with this, we define a function that takes in each combination of pokemon types (every column of `combinations`), and sums the number of types the combination is super effective against. We find the types that appear in the most number of "best combinations".


```R
super_effective_nb <- function(indices){
#   for each pokemon group (6-row-subset of super_effective_m), take the column sum for each of the 18 columns. 
#   If colSums > 0, then there is at least 1 of the 6 types that is super effective against the type, so count 
#   this as a 1. Sum all types your group is super effective against.
  sum(colSums(super_effective_m[indices, ]) > 0)
}
super_effective_results <- apply(combinations, 2, super_effective_nb) #find the number of super-effectives for each possible combination
best_combos <- combinations[, super_effective_results == max(super_effective_results)] #10 out of 18564
strongest_teams <- matrix(rownames(super_effective_m)[best_combos], nrow = 6)
strongest_teams %>%
  data.frame() %>%
  mutate(count = 1:6) %>%
  pivot_longer(-count, names_to = 'group', values_to = 'type') %>%
  select(-count) %>%
  group_by(type) %>%
  summarise(count = length(unique(group))) %>%
  arrange(desc(count))
```
    
<table class="table table-striped" style="width: auto !important; ">
<thead>
	<tr><th scope=col>type</th><th scope=col>count</th></tr>
</thead>
<tbody>
	<tr><td>Ground  </td><td>10</td></tr>
	<tr><td>Fighting</td><td> 8</td></tr>
	<tr><td>Flying  </td><td> 8</td></tr>
	<tr><td>Ice     </td><td> 8</td></tr>
	<tr><td>Dark    </td><td> 5</td></tr>
	<tr><td>Ghost   </td><td> 5</td></tr>
	<tr><td>Grass   </td><td> 4</td></tr>
	<tr><td>Poison  </td><td> 4</td></tr>
	<tr><td>Electric</td><td> 2</td></tr>
	<tr><td>Fairy   </td><td> 2</td></tr>
	<tr><td>Rock    </td><td> 2</td></tr>
	<tr><td>Steel   </td><td> 2</td></tr>
</tbody>
</table>


We can now see the pokemon types that are needed for the maximum possible super-effective team combination. (you cannot run away from having a ground type)

### Equilibrium

Let's assume this whole exercise is correct up to this point (ignore distribution of super-effective attacks, super-effective attacks worth 4x normal attacks, etc.). If we assume that this "optimal" response will be played, is the Nash equilibrium to reply with this? For illustration, I will just use the first group identified


```R
# Picking one of 10 the strongest combinations:
strongestTeamIndex <- unique(row(super_effective_m)[rownames(super_effective_m) %in% strongest_teams[,1]])
super_effective_m[,strongestTeamIndex]
```

<table class="table table-striped" style="width: auto !important; ">
<thead>
	<tr><th></th><th scope=col>Electric</th><th scope=col>Ice</th><th scope=col>Fighting</th><th scope=col>Ground</th><th scope=col>Flying</th><th scope=col>Ghost</th></tr>
</thead>
<tbody>
	<tr><th scope=row>Normal</th><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Fire</th><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Water</th><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Electric</th><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td></tr>
	<tr><th scope=row>Grass</th><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Ice</th><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td></tr>
	<tr><th scope=row>Fighting</th><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Poison</th><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Ground</th><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Flying</th><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Psychic</th><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Bug</th><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Rock</th><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>0</td></tr>
	<tr><th scope=row>Ghost</th><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td></tr>
	<tr><th scope=row>Dragon</th><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Dark</th><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td></tr>
	<tr><th scope=row>Steel</th><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
	<tr><th scope=row>Fairy</th><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td></tr>
</tbody>
</table>




```R
super_effective_nb_subset <- function(indices){
    sum(colSums(super_effective_m[indices, strongestTeamIndex]) > 0)
}
super_effective_results_subset <- apply(combinations, 2, super_effective_nb_subset)
best_combos_subset <- combinations[, super_effective_results_subset == max(super_effective_results_subset)]
dim(best_combos_subset)
```
> 6 x 396

Clearly, when you have a fixed team of only 6 to counter, there are a lot more possible "optimal" combinations. The point is to see if you have the optimal 10 among the 396 combinations identified.


```R
subset <- t(best_combos_subset) %>%
  as.data.frame() %>%
  unite('string', V1:V6)
optimal <- t(best_combos) %>%
  as.data.frame() %>%
  unite('string', V1:V6)
  
optimal$$string %in% subset$$string
```
> TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE 

All 10 appear in the subset! Which suggests that, ceteris paribus, it is always better to choose among the 10 optimal strategies identified above that maximise super effective attacks when responding to a `c(Electric, Ice, Fighting, Ground, Flying, Ghost)` team. 

<b id="f1">1</b> $$\frac{18!}{12! \cdot 6!} = 18564$$ [↩](#a1)
