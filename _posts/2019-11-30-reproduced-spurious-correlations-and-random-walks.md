---
layout: post
title: Spurious Correlations and Random Walks
categories: []
tags: []
description: Impact of random walks on hypothesis testing for time series
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
library(knitr)
library(IRdisplay)
library(ggpubr)
```
-->

*Reproduced from [Fabian Dablander](https://fabiandablander.com/r/Spurious-Correlation.html)*

### Introduction

This short post will explore the implications of random walk processes in inducing spurious correlations and false positives in hypothesis testing. 

Recall that an AR series generally takes the following form:  
<br>
<center> $$Y_{t} = \phi Y_{t-1} + \epsilon_{t} - (1) $$ </center>
<br>
where $$\phi$$ measures the autocorrelation, and $$\epsilon \sim \mathbb{N}(0,\sigma^{2})$$. Note that a random walk is just a special case of the AR process where $$\phi$$ = 1.
<br>
<br>

### The randomness in random walks

To explore the impact of $$\phi$$ in generating data, we build a function that simulates a data generating process given values of $$n$$, $$\phi$$ and $$\sigma$$ where $$n$$ determines the number of datapoints to generate; $$\phi$$ is the first order autocorrelation, and $$\sigma$$ is the variance of the errors in $$\epsilon$$.

```R
simulate_ar <- function(n, phi, sigma = .1){
    y <- rep(0, n)
    for (t in seq(2, n)){
        y[t] <- phi*y[t-1] + rnorm(1, 0, sigma)
    }
    y
}
```

```R
n <- 100
set.seed(1)
 
rw1 <- simulate_ar(n, phi = 1)
rw2 <- simulate_ar(n, phi = 1)
rw3 <- simulate_ar(n, phi = 1)

ar1 <- simulate_ar(n, phi = 0.5)
ar2 <- simulate_ar(n, phi = 0.5)
ar3 <- simulate_ar(n, phi = 0.5)
```

```R
options(repr.plot.width = 10, repr.plot.height = 6)
combinedPlot <- data.frame(
  Time = rep(seq.int(1,100), 3), 
  Values = c(ar1, ar2, ar3, rw1, rw2, rw3),
  Group = rep(c('AR_1', 'AR_2', 'AR_3', 'RW_1', 'RW_2', 'RW_3'), each = 100)
  ) %>% 
  mutate(Group = factor(Group))

combinedPlot %>%
  ggplot(aes(x = Time, y = Values,  colour = `Group`)) + 
  geom_line() +
  geom_point() + 
  theme_classic() + 
  scale_colour_manual(values = c('#6C6B74', '#2E303E', '#212624', '#F22F08', '#FAAB00', '#2721DB')) +
  theme(text = element_text(size=20))
```

As an example, let's generate 3 AR(1) processes and 3 random walks and chart them. Notice how much better behaved the AR processes are compared to the random walk. Looking at Figure 1, it's clear that the random walk processes are much more erratic than the AR processes ($$\phi$$ = 0.5), also called "non-stationary" in statistical gobbledygook.
    
![png](/images/2019-11-30-reproduced-spurious-correlations-and-random-walks_files/2019-11-30-reproduced-spurious-correlations-and-random-walks_7_0.png)
    
### Why should anyone care about this?

For one, randomness means that any correlations computed with random walks are unreliable (jargon: spurious). This is clear from the correlation tables below - the correlation between random walk series clearly exceed those seen in the AR1.

```R
library(kableExtra)
t1 <- round(cor(cbind(rw_series_1 = rw1, rw_series_2 = rw2, rw_series_3 = rw3)), 2)
t2 <- round(cor(cbind(ar_series_1 = ar1, ar_series_2 = ar2, ar_series_3 = ar3)), 2)
table_1 <- t1 %>%
    kable('html', caption = 'Table 1: Random Walk Corr') %>%
    kable_styling(bootstrap_options = "striped", full_width = F, position = "left") %>%
    as.character() %>%
    display_html()

table_2 <- t2 %>%
    kable('html',caption = 'Table 2: AR1 Corr') %>%
    kable_styling(bootstrap_options = "striped", full_width = F, position = "left") %>%
    as.character() %>%
    display_html()
```

<table class="table table-striped" style="width: auto !important; ">
<caption>Table 1: Random Walk Corr</caption>
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> rw_series_1 </th>
   <th style="text-align:right;"> rw_series_2 </th>
   <th style="text-align:right;"> rw_series_3 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> rw_series_1 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> -0.49 </td>
   <td style="text-align:right;"> -0.29 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rw_series_2 </td>
   <td style="text-align:right;"> -0.49 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 0.59 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rw_series_3 </td>
   <td style="text-align:right;"> -0.29 </td>
   <td style="text-align:right;"> 0.59 </td>
   <td style="text-align:right;"> 1.00 </td>
  </tr>
</tbody>
</table>



<table class="table table-striped" style="width: auto !important; ">
<caption>Table 2: AR1 Corr</caption>
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> ar_series_1 </th>
   <th style="text-align:right;"> ar_series_2 </th>
   <th style="text-align:right;"> ar_series_3 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> ar_series_1 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> -0.06 </td>
   <td style="text-align:right;"> -0.03 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ar_series_2 </td>
   <td style="text-align:right;"> -0.06 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 0.06 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ar_series_3 </td>
   <td style="text-align:right;"> -0.03 </td>
   <td style="text-align:right;"> 0.06 </td>
   <td style="text-align:right;"> 1.00 </td>
  </tr>
</tbody>
</table>


This means that when you're doing any sort of time series comparison (say, you're day trading and you find an awesome correlation that you can't wait to arbitrage), you might find it hard to be sure if the correlation you're seeing is real, or just 2 random walks being chummy.

### How is actual hypothesis testing affected?

First, we test the effect of different time-series lengths on the validity of the hypothesis test. By simulating 100 draws each of 2 random walks with 50/100/200/500/1000/2000 observations, we compute the average correlation, average tstat, and average proportion of the 100 observations that pass the significance test at some fixed significance level (5% in this case).

```R
set.seed(1)
 
times <- 100 #number of times you want to simulate the corr between the ARs
ns <- c(50, 100, 200, 500, 1000, 2000) #number of observations for each AR process
 
comb <- expand.grid(times = seq(times), n = ns)

res <- matrix(NA, nrow = nrow(comb), ncol = 5)
colnames(res) <- c('ix', 'n', 'cor', 'tstat', 'pval')

for (i in seq(nrow(comb))){ #for eah row in the cross product of times and ns
  n <- comb[i, 2] #assign count of observations to dataframe column
  ix <- comb[i, 1] #assign the index number of the simulation to data frame
  test <- cor.test(simulate_ar(n, phi = 1), simulate_ar(n, phi = 1)) #find the correlation between 2 simulated time series of length n
  res[i, ] <- c(ix, n, test$$estimate, test$$statistic, test$$p.value) #store in the res dataframe
}
 
tab <- data.frame(res) %>% 
  group_by(n) %>% #group by the lengths of the data
  summarize(
    avg_abs_corr = mean(abs(cor)), #for the average correlation among the 100 runs for each data length
    avg_abs_tstat = mean(abs(tstat)), #find the average t-stat among the 100 runs for each data length
    percent_sig = mean(pval < .05) #find the 
  ) %>% 
  data.frame() %>%
#   mutate(percent_sig = paste0(as.character(percent_sig*100), '%')) %>%
  `colnames<-`(c('Count of Observations', 'Average Absolute Correlation', 'Average Absolute T-Stat', 'Percent of observations that pass the significance test at 5%'))
  
 
round(tab, 2)
```

<table class="table table-striped" style="width: auto !important; ">
<thead>
	<tr><th scope=col>Obs Count</th><th scope=col>Average Corr</th><th scope=col>Average tstat</th><th scope=col>% of significant obs at 5%</th></tr>
</thead>
<tbody>
	<tr><td>  50</td><td>0.41</td><td> 3.57</td><td>0.71</td></tr>
	<tr><td> 100</td><td>0.46</td><td> 6.58</td><td>0.85</td></tr>
	<tr><td> 200</td><td>0.45</td><td> 8.88</td><td>0.85</td></tr>
	<tr><td> 500</td><td>0.37</td><td>10.63</td><td>0.86</td></tr>
	<tr><td>1000</td><td>0.41</td><td>17.05</td><td>0.88</td></tr>
	<tr><td>2000</td><td>0.39</td><td>23.39</td><td>0.97</td></tr>
</tbody>
</table>

While the average random walk correlation basically stays the same regardless of the length of the time series, notice how the average t-stat increases, and a larger and larger proportion of your t-tests comes back significanct (97% at 2000 obs!!) **despite the fact that these were 2 random walk series!**

Instead of using the standardised coefficient, let's convert this to an equivalent but more commonly encountered form.<sup id="a1">[1](#f1)</sup> What happens when we try to draw conclusions using $$\beta_{1}$$ from the regression $$ts_{0} = \alpha + \beta_{1} \ast ts_{1} + \epsilon$$ where $$ts_{0}$$ and $$ts_{1}$$ are 2 random walks?

```R
# Simulate 2 ARs and find the regression coef between them
regress_ar <- function(n, phi, sigma){
  y <- simulate_ar(n, phi, sigma)
  x <- simulate_ar(n, phi, sigma)
  return(coef(summary(lm(y ~ x)))[2, c(1, 2, 3)])
}
 
bootstrap_limit <- function(ns, phi, sigma, times = 200) {
  #Create empty matrix that records 3 values; simulation index n, regression coefficient, and 
  #t-stat of regression coef
  res <- matrix(NA, nrow = times * length(ns), ncol = 4)
  colnames(res) <- c('n', 'b1', 'std.error', 'tstat')
 
  ix <- 1
  for (n in ns) { #Looping across the 6 series lengths
    for (i in seq(times)) { #Generate `times` number of runs for each series length
      coefs <- regress_ar(n, phi, sigma)
      res[ix, ] <- c(n, coefs)
      ix <- ix + 1
    }
  }
  return(data.frame(res))
}
 
set.seed(1)
ns <- c(100, 200, 500, 1000, 2000)
res_ar <- bootstrap_limit(ns, .5, .1)
res_rw <- bootstrap_limit(ns,  1, .1)
```


```R
# Chart for t-stat
t1 <- res_ar %>%
  ggplot(aes(x = factor(n), y = tstat)) +
  geom_boxplot(outlier.shape = NA) +
  theme_classic() +
  ggtitle('t-stat of B1 (AR) with different lengths') +
  xlab('Length of Time Series')

t2 <- res_rw %>%
  ggplot(aes(x = factor(n), y = tstat)) +
  geom_boxplot(outlier.shape = NA) +
  theme_classic() +
  ggtitle('t-stat of B1 (random walk) with different lengths') +
  xlab('Length of Time Series')

ggpubr::ggarrange(t1, t2)
```

    
![png](/images/2019-11-30-reproduced-spurious-correlations-and-random-walks_files/2019-11-30-reproduced-spurious-correlations-and-random-walks_15_0.png)
    


It should be quite clear from the figure above that something weird occurs when the usual regression is done on a random walk! Compared to the AR(1) process, the t-statistic for $$\beta$$ term in the random walk regression blows up as the series length increases! This isn't the most intuitive result, so let's do a bit more digging for the culprit. Recall that the test statistic used here is $$t = Z/s = \frac{\overline{X} - \mu}{\hat{\sigma}/\sqrt{n}}$$. 


```R
# Chart for beta
t1 <- res_ar %>%
  ggplot(aes(x = factor(n), y = b1)) +
  geom_boxplot(outlier.shape = NA) +
  theme_classic() +
  ggtitle('B1 of AR with different lengths') +
  xlab('Length of Time Series')

t2 <- res_rw %>%
  ggplot(aes(x = factor(n), y = b1)) +
  geom_boxplot(outlier.shape = NA) +
  theme_classic() +
  ggtitle('B1 of random walks with different lengths') +
  xlab('Length of Time Series')

ggpubr::ggarrange(t1, t2)
```

    
![png](/images/2019-11-30-reproduced-spurious-correlations-and-random-walks_files/2019-11-30-reproduced-spurious-correlations-and-random-walks_17_0.png)
    

It's clear that the issues lies with the difference in the $$\beta$$ estimate as the length of the series increases. On the left, the $$\beta$$ estimate for the AR converges as the series length increases. Intuitively, your distribution becomes narrower with more observations since your unbiased mean converges to the true population mean. This is not the case with the random walk; the distribution of the $$\beta$$ term stays constant regardless of observation length.

```R
# Chart for t-stat
t1 <- res_ar %>%
  mutate(std.error.dividedSampleSize = std.error/sqrt(n)) %>%
  ggplot(aes(x = factor(n), y = std.error.dividedSampleSize)) +
  geom_boxplot(outlier.shape = NA) +
  theme_classic() +
  ggtitle('std.error/sqrt(n) of B1 (AR) with different lengths') +
  theme(plot.title = element_text(size = 11)) +
  xlab('Length of Time Series')

t2 <- res_rw %>%
  mutate(std.error.dividedSampleSize = std.error/sqrt(n)) %>%
  ggplot(aes(x = factor(n), y = std.error.dividedSampleSize)) +
  geom_boxplot(outlier.shape = NA) +
  theme_classic() +
  ggtitle('std.error/sqrt(n) of B1 (random walk) with different lengths') +
  theme(plot.title = element_text(size = 11)) +
  xlab('Length of Time Series')

ggpubr::ggarrange(t1, t2)
```
    
![png](/images/2019-11-30-reproduced-spurious-correlations-and-random-walks_files/2019-11-30-reproduced-spurious-correlations-and-random-walks_19_0.png)
    


Mitigating this, the $$\hat{\sigma}/\sqrt n$$of the mean does not decrease to the same extent for the random walk. Nonetheless, the effect on the $$\beta$$ coefficient estimate dominates.

Overall, the conclusion from this is that hypothesis tests on random walk regressions will almost definitely bias your results horrendously, and to make things worse, the problem is worse with more data.

### Conclusion: t-test for random walk processes will give you many false rejections of the null! 

Does this mean that ALL time series regressions are susceptible to this problem? Well yes, but the extent differs. The nearer your AR term is to a random walk, the more this becomes a problem.


```R
set.seed(1)
 
n <- 200
times <- 100
phis <- seq(0, 1, .01)
comb <- expand.grid(times = seq(times), n = n, phis)
ncomb <- nrow(comb)
 
res <- matrix(NA, nrow = ncomb, ncol = 6)
colnames(res) <- c('ix', 'n', 'phi', 'cor', 'tstat', 'pval')
 
for (i in seq(ncomb)) {
  ix <- comb[i, 1]
  n <- comb[i, 2]
  phi <- comb[i, 3]
  
  test <- cor.test(simulate_ar(n, phi = phi), simulate_ar(n, phi = phi))
  res[i, ] <- c(ix, n, phi, test$$estimate, test$$statistic, test$$p.value)
}
 
dat <- data.frame(res) %>% 
  group_by(phi) %>% 
  summarize(
    avg_abs_corr = mean(abs(cor)),
    avg_abs_tstat = mean(abs(tstat)),
    percent_sig = mean(pval < .05)
  )

dat %>%
  ggplot(aes(x = phi, y = avg_abs_corr)) +
  geom_point() +
  xlab('AR1 Coefficient') +
  ylab('Average Absolute Correlation')
```
    
![png](/images/2019-11-30-reproduced-spurious-correlations-and-random-walks_files/2019-11-30-reproduced-spurious-correlations-and-random-walks_21_1.png)

As your AR coefficient nears 1, the abs correlation between different series increases in **magnitude**...


```R
dat %>%
  ggplot(aes(x = phi, y = avg_abs_tstat)) +
  geom_point() +
  xlab('AR1 Coefficient') +
  ylab('Average Absolute T-Stat')
```


    
![png](/images/2019-11-30-reproduced-spurious-correlations-and-random-walks_files/2019-11-30-reproduced-spurious-correlations-and-random-walks_23_0.png)
    


...and also in **t-stat**...



```R
dat %>%
  ggplot(aes(x = phi, y = percent_sig)) +
  geom_point() +
  xlab('AR1 Coefficient') +
  ylab('Percentage of correlated series at 5% significance')
```


    
![png](/images/2019-11-30-reproduced-spurious-correlations-and-random-walks_files/2019-11-30-reproduced-spurious-correlations-and-random-walks_25_0.png)
    


...leading to a much larger proportion of false positives!

<b id="f1">1</b> Note that the regression coefficient is NOT the same as the  simple correlation. Recall that in a $$y = \alpha + \beta x + \epsilon$$ setting, $$\beta = cor(x,y)\cdot \frac{SD(y)}{SD(x)}$$ [↩](#a1)
