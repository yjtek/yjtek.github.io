---
layout: post
title: The Inspection Paradox
categories: []
tags: []
description: Why does the bus always seem to take longer than the advertised waiting time?
published: true
fullview: false
comments: true
author: Yong Jian
---

*Based on a blog post by [Jake VanderPlas](http://jakevdp.github.io/blog/2018/09/13/waiting-time-paradox/)*

### Introduction

Bit of a lighter post after the MCMC series. Came across this article on the waiting time paradox, and I thought it would be a good chance to dosomething light-hearted amidst this COVID outbreak.

We all know that gnawing irritation we feel when, somehow, things are not as rosy as advertised. Bus schedules say that buses come every 10 minutes on average, but why does it always feel like you just missed the bus? Schools say that their average class size is around 20, but how do you always end up in a sardine-packed class? Why do your friends on Facebook always have more friends than you do?

![jpg](/images/2020-04-02-the-inspection-paradox_files/Singapore-Bus-Stop-Poster-Guide.jpg)
<!-- <span style="color: grey; font-style: italic;">Lies</span> -->

Welcome to the inspection paradox, or as the cited post puts it, the "why is my bus always late" paradox.

### What is the inspection paradox?
The inspection paradox occurs when the probability of observing a quantity is related to the quantity being observed. This definition is pretty convoluted on it's own, so let's talk about an intuitive example - class size.

Imagine a university cohort of 100 students with 3 different classes in each semester. Class 1 has 70 students, class 2 has 20, and class 3 has 10. From the university's perspective, the average class size is a straightforward $$\frac{100}{3} = 33$$. 

What happens if the students are surveyed? Now the story is completely different! By definition, the class size of 70 is experienced by 70 students, which biases the sample average! Instead of $$\frac{100}{3} = 33$$, we now have a weighted average of class size $$(0.7 \cdot 70) + (0.2 \cdot 20) + (0.1 \cdot 10) = 54$$ instead.

In other words, the probability of an individual observing a class size of 70 is directly related to the probability of him/her being in that class!

### Seeing this in practice
Let's see this for ourselves using a very relatable example: bus arrival timings. First, let's simulate the arrival time of 100000 bus arrivals with 10 minutes as the average waiting time between buses. The timeline below shows the first 10 simulated bus arrivals assuming the arrivals are random. 


```python
import numpy as np
import matplotlib.pyplot as plt
simulated_bus_count = 100000
time_between_buses = 10
random = np.random.RandomState(1)
bus_arrival_times = simulated_bus_count * time_between_buses * np.sort(random.random(simulated_bus_count))

fig, ax = plt.subplots()
plt.plot(bus_arrival_times[:10], [0]*10, '-o', color = 'blue')
ax.spines['right'].set_visible(False)
ax.spines['top'].set_visible(False)
ax.spines['left'].set_visible(False)
tmp = 1
for i in range(10):
    ax.annotate(round(bus_arrival_times[i], 1), (bus_arrival_times[i], 0.01*tmp))
    tmp *= -1
plt.yticks([])
plt.title(f'Time of Bus Arrivals ({time_between_buses} minute average)')
plt.show()
```

    
![png](/images/2020-04-02-the-inspection-paradox_files/2020-04-02-the-inspection-paradox_1_0.png)
    


Great! Let's just make sure that, on average, the time between arrivals is 10 minutes.


```python
np.diff(bus_arrival_times).mean()
```
> 9.999899149741081



Next, we simulate the arrival of passengers randomly along this range. 


```python
random = np.random.RandomState(123)
num_passengers = 100000

# Get passenger arrival time
passenger_arrival_times = simulated_bus_count * time_between_buses * np.sort(random.random(num_passengers))

fig, ax = plt.subplots()
plt.plot(bus_arrival_times[:20], [0]*20, '-o', color = 'blue')
plt.plot(passenger_arrival_times[:10], [0]*10, '-o', color = 'red')
ax.spines['right'].set_visible(False)
ax.spines['top'].set_visible(False)
ax.spines['left'].set_visible(False)
tmp = 1
for i in range(0, 20, 3):
    ax.annotate(round(bus_arrival_times[i], 1), (bus_arrival_times[i], 0.01*tmp))
    ax.annotate(round(passenger_arrival_times[i], 1), (bus_arrival_times[i], 0.01*-tmp))
plt.yticks([])
plt.title(f'Time of Bus and Passenger Arrivals')
plt.show()
```


    
![png](/images/2020-04-02-the-inspection-paradox_files/2020-04-02-the-inspection-paradox_5_0.png)
    


We overlay our randomly generated passengers on our bus timeline. Using the np.searchsorted() function, we input the array of bus and passenger arrivals, and the function returns an `index` of bus_arrival_times for which inserting each passenger_arrival_times[i] preserves the ordering of the array. Since side = 'right', it returns the largest possible value of the index, or the nearest possible bus arrival for each passenger. Using this, we compute the average waiting time for each passenger.


```python
i = np.searchsorted(bus_arrival_times, passenger_arrival_times, side='right')
average_waiting_time = (bus_arrival_times[i] - passenger_arrival_times).mean()
print(average_waiting_time)
```
> 10.061306085620576


This is a surprising result! Although the average time between buses is 10 minutes, passengers experience a 10 minute waiting time on average instead of 5mins, which you would expect if you can arrive randomly at any time between 2 buses.

Like the class size example above, this result arises because you have a higher chance of 'catching' in a time period where the time between buses is large (since it takes up a longer stretch of the number line), and this biases average waiting time upwards!

### Conclusion
This was a really interesting result to me, because it's a prime example of how our statistical heuristics can end up working horrendously. Yes, I know that bus arrivals don't actually follow a uniform distribution, but I found this instructive an example to illustrate the problem of our heuristics in statistical reasoning.
