---
layout: post
title: Finding Prime Divisors
categories: []
tags: []
description: How to find all prime divisors given a number
published: true
fullview: false
comments: true
author: Yong Jian
---

*Problem from [edabit](https://edabit.com/challenge/FbQqXepHC4wxrWgYg).*

### Introduction

Have been trying to work through one problem a day as an exercise to better learn algorithms and data structures, and I'll periodically put up interesting problems here with my own solution.

### Problem

**Prime Divisors**

Given a number, return all its prime divisors in a list. Create a function that takes a number as an argument and returns all its prime divisors.

- n = 27
- All divisors: [3, 9, 27]
- Finally, from that list of divisors, return the prime ones: [3]


**Examples**

`prime_divisors(27)`
> [3]

`prime_divisors(99)`
> [3, 11]

`prime_divisors(3457)`
>[3457]

### Solution

Interestingly, this problem comes with some mathematical baggage. Since we want only prime divisors, we need a way to identify primes iteratively for a given value.

A simple way is to use a well-known algorithm called the "Sieve of Eratosthenes". Basically, you start with the known primes (2 and 3), and iteratively eliminate their multiples as primes. As we iterate through the list of numbers, we will eventually end up with a list with only primes. I've tried to implement this below in the `get_all_primes_till(n)` function.


```python
def get_all_primes_till(n):
    '''
    Implement Sieve of Eratosthenes.
    '''
    
    full_number_list = [x+1 for x in range(n)] #Generate a running list of numbers ranging from 1 to the user input value
    is_prime = [True] * n #For each number in `full_number_list`, set is_prime status to True
    max_list_value = n #create a variable for the maximum value in the list

    for index, number in enumerate(full_number_list): #for each number in the list, 
        if (is_prime[index]) & (number != 1): #if the number is prime and the number is not 1
            is_prime[index::number] = [False for i in is_prime[index::number]] #set all multiples of the number to False in the `is_prime` list
            is_prime[index] = True #the number itself is still a prime
        else: #if the number is not prime OR the number is 1, skip
            continue
    
    return [full_number_list[i] for i, x in enumerate(is_prime) if x] #return a list of numbers that are not enumerate
```

Once you have a function that gives you a list of primes, finding divisors simply means making use of the modulo operation in Python. 


```python
def prime_divisors(num):
    primes_till_input = get_all_primes_till(num)
    prime_divisors = []
    for prime in primes_till_input:
        if num % prime == 0:
            prime_divisors.append(prime)
    return [prime_divisor for prime_divisor in prime_divisors if prime_divisor != 1]
```

Testing out the functions below, we see that it works as intended though this is probably not the fastest implementation.


```python
prime_divisors(20)
```




    [2, 5]




```python
%timeit prime_divisors(20)
```

    6.19 µs ± 66.5 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)



```python
%timeit prime_divisors(200)
```

    40.5 µs ± 329 ns per loop (mean ± std. dev. of 7 runs, 10000 loops each)



```python
%timeit prime_divisors(2000)
```

    444 µs ± 7.44 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)

