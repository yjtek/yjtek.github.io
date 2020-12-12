---
layout: post
title: Python Practise Problems
categories: []
tags: []
description: Practise problems with Python
published: true
fullview: false
comments: true
author: Yong Jian
---


*Problems from [edabit](https://edabit.com/challenges).*

### Introduction

Have been trying to work through one problem a day as an exercise to better learn algorithms and data structures, and I'll periodically put up interesting problems here with my own solution.

### Problem 1: Prime Divisors

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

#### Solution

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


### Problem 2: Longest Alternating Substrings

Given a string of digits, return the longest substring with alternating odd/even or even/odd digits. If two or more substrings have the same length, return the substring that occurs first.


`longest_substring("225424272163254474441338664823")`
> "272163254" (substrings = 254, 272163254, 474, 41, 38, 23)

`longest_substring("594127169973391692147228678476")`
> "16921472" (substrings = 94127, 169, 16921472, 678, 476)

`longest_substring("721449827599186159274227324466")`
> "7214" (substrings = 7214, 498, 27, 18, 61, 9274, 27, 32. 7214 and 9274 have same length, but 7214 occurs first.)

#### Solution

No real trick here. The main issue is getting Python to recognise when to append the alternating values, and when to end the substring. In the implementation below (not optimal), this is done by building a string of indices if the values are alternating, and appending a separating string when it is not. 


```python
def longest_substring(string):
    string_to_list = list(string)
    odd_even_indicator = [str(int(x) % 2) for x in string_to_list]
    joined_string = ''.join(odd_even_indicator)
    joined_string_ahead = joined_string[1:]

    alternating_substring_positions = []

    for i in range(len(joined_string)):
        if i == (len(joined_string)-1):
            if (joined_string[-1] == joined_string[-2]):
                alternating_substring_positions.append('~~')
                continue
            else:
                alternating_substring_positions.append(str(i) + ',')
                continue

        if joined_string[i] != joined_string_ahead[i]:
            alternating_substring_positions.append(str(i) +',')
        else:
            alternating_substring_positions.append(str(i) +',')
            alternating_substring_positions.append('~~')

    split_list = ''.join([str(x) for x in alternating_substring_positions]).split('~')
    split_list = [val for val in split_list if val != '']
    split_list = [val.split(',') for val in split_list]
    length_alternating = [len(sublist)-1 for sublist in split_list]
    max_alternating_sequence_index = split_list[length_alternating.index(max(length_alternating))]
    max_alternating_sequence_index = max_alternating_sequence_index[:-1]

    return(''.join([string[int(index)] for index in max_alternating_sequence_index]))

longest_substring("225424272163254474441338664823")
```




    '272163254'



### Problem 3: Josephus Permutation

A group of n prisoners stand in a circle awaiting execution. Starting from an arbitrary position(0), the executioner kills every kth person until one person remains standing, who is then granted freedom (see examples).

Create a function that takes 2 arguments — the number of people to be executed n, and the step size k, and returns the original position (index) of the person who survives.

`who_goes_free(9, 2)`
> 2

> Prisoners = [0, 1, 2, 3, 4, 5, 6, 7, 8] <br>
> Executed people replaced by - (a dash) for illustration purposes. <br>
> 1st round of execution = [0, -, 2, -, 4, -, 6, -, 8]  -> [0, 2, 4, 6, 8] <br>
> 2nd round = [-, 2, -, 6, -] -> [2, 6]  # 0 is killed in this round because it's beside 8 who was skipped over.  <br> 
> 3rd round = [2, -]  <br>

`who_goes_free(9, 3)`
> 0

> [0, 1, 2, 3, 4, 5, 6, 7, 8] <br>
> [0, 1, -, 3, 4, -, 6, 7, -] -> [0, 1, 3, 4, 6, 7] <br>
> [0, 1, -, 4, 6, -] -> [0, 1, 4, 6] <br>
> [0, 1, -, 6] -> [0, 1, 6]  <br>
> [0, -, 6] -> [0, 6]  <br>
> [0, -] -> [0]

#### Solution

The catch here is how to tell Python to iterate across individuals EXCLUDING ELIMINATED INDIVIDUALS. I've implemented it simply by rearranging across cycles (i.e. once we go a complete circle, the list is rearranged in the order of the last eliminated person). This solves the problem, though there are probably some more efficient data structures to handle this. You can verify the calculator below against this [site](http://webspace.ship.edu/deensley/mathdl/Joseph.html).


```python
def who_goes_free(n, k):
    prisoners = [x for x in range(n)]
    while len(prisoners) > 1:
        if len(prisoners) < k:
            executed = k % len(prisoners) - 1
            prisoners[executed] = '~'
        else:
            prisoners[(k-1)::k] = ['~' for prisoner in prisoners[(k-1)::k]]

        last_executed = -prisoners[::-1].index('~')
        prisoners = prisoners[last_executed:] + prisoners[:last_executed]
        prisoners = [prisoner for prisoner in prisoners if prisoner != '~']

    return prisoners

who_goes_free(12, 5)
```




    [0]


