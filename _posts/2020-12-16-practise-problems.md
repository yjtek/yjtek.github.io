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

### Problem 1: Crop Fields

You're given a 2D list / matrix of a crop field. Each crop needs a water source. Each water source hydrates the 8 tiles around it. With "w" representing a water source, and "c" representing a crop, is every crop hydrated?

`crop_hydrated([[ "w", "c" ], [ "w", "c" ], [ "c", "c" ]])`
> True

`crop_hydrated([[ "c", "c", "c" ]])`
> False. there isn't even a water source.

`crop_hydrated([[ "c", "c", "c", "c" ], [ "w", "c", "c", "c" ], [ "c", "c", "c", "c" ], [ "c", "w", "c", "c" ]])`
> False

Note that "w" on its own should return True, and "c" on its own should return False.

#### Solution

Quite a simple solution, just looping through every possible position and checking if it is hydrated. Key is noting that hydration is simply every element that is (-1, -1) to (+1, +1) away from each water source.


```python
def crop_hydrated(field):
    
    # Get all water positions
    water_positions = []
    for row, sublist in enumerate(field):
        water_positions.append([(row, col) for col, letter in enumerate(sublist) if letter == 'w'])
    water_positions = [subtuple for sublist in water_positions for subtuple in sublist if sublist != []]
    
    # Get a list of all position combinations
    nrow = len(field)
    ncol = len(field[0])
    all_positions = []
    for i in range(nrow):
        for j in range(ncol):
            all_positions.append((i,j))

    # Get positions that are hydrated with water source
    covered_positions = []
    for water_position in water_positions:
        rowpos = water_position[0]
        colpos = water_position[1]
    
        covered_row_min = max(rowpos-1, 0)
        covered_row_max = min(rowpos+1, nrow-1)
        covered_col_min = max(colpos-1, 0)
        covered_col_max = min(colpos+1, ncol-1)

        for i in range(covered_row_min, covered_row_max+1):
            for j in range(covered_col_min, covered_col_max+1):
                covered_positions.append((i,j))
    
    # If a position is in the all_positions, but not in covered position, return it. Return boolean chceking if list is empty
    return [position for position in all_positions if position not in covered_positions] == []
```


```python
crop_hydrated([[ "w", "c" ], [ "w", "c" ], [ "c", "c" ]])
```




    True



### Problem 2: Numbers First, Letters Second

Write a function that sorts list while keeping the list structure. Numbers should be first then letters both in ascending order.

`num_then_char([[1, 2, 4, 3, "a", "b"], [6, "c", 5], [7, "d"], ["f", "e", 8]])`
> [[1, 2, 3, 4, 5, 6], [7, 8, "a"], ["b", "c"], ["d", "e", "f"]]

num_then_char([[1, 2, 4.4, "f", "a", "b"], [0], [0.5, "d","X",3,"s"], ["f", "e", 8], ["p","Y","Z"], [12,18]])
> [[0, 0.5, 1, 2, 3, 4.4], [8], [12, 18, "X", "Y", "Z"], ["a", "b", "d"], ["e", "f", "f"], ["p", "s"]]

Test cases will contain integer and float numbers and single letters.

#### Solution

The solution here relies on the key assumption that the sublists are not nested beyond 2 levels, and singleton values are also wrapped in a list.

Solving this requires 2 steps. The first is to figure out the structure of the base list, so we know how to maintain it. The second is to find a way to sort the elements, so you can feed it into the appropriate structure.

To sort the elements, we simply want to flatten the list, and separate them into two `number` and `character` lists. Ths allows us to use the built in `sort()` function, and concatenating the results will give us the order that we want. We can simply iterate over this to fit the structure of the original list.


```python
lst = [[2,1,4,3], ['z'], ['c', 'b','d','a']]

def num_then_char(lst):
    import numpy as np

    # # def num_then_char(lst):
    flat_list = sum(lst, []) #sums each element in lst with the empty list
    char_list = sorted([x for x in flat_list if isinstance(x, str)])
    int_list = sorted([x for x in flat_list if isinstance(x, int)])
    flat_list = int_list + char_list

    list_sizes = list(map(len, lst))
    list_sizes_cumsum = list(np.cumsum(list_sizes))
    list_sizes_cumsum = [0] + list_sizes_cumsum
    return_sorted = []
    for i in range(0, (len(list_sizes_cumsum)-1)):
        return_sorted.append(flat_list[list_sizes_cumsum[i]:list_sizes_cumsum[i+1]])
    return(return_sorted)
        
num_then_char(lst)
```

    [[1, 2, 3, 4], ['a'], ['b', 'c', 'd', 'z']]



### Problem 3: Number of deletions

Write a function `def solution(S)` that, given a string S consisting of N lowercase letters, returns the minimum number of letters that must be deleted to obtain a word in which every letter occurs a unique number of times. We only care about occurrences of letters that appear at least once in the result.

`solution('aaaabbbb')`
> 1 (Delete 1 instance of a or b to give you a: 4 and b: 3

`solution('ccaaffddecee')`
> 6 (Delete all occurrences of e and f and one occurrence of d to obtain the word `ccaadc`. Doesn't matter that some occurrences are 0)

`solution('eeee')` 
> 0 (No deletions needed, since only 1 letter)

`solution('example')`
> 4

Assume N $\epsilon$ [0, 300,000] and S $\epsilon$ [a-z]

#### Solution

The key here is realising that collections.Counter() gives you a wonderful and fast way to count the elements in your string, and can be cast as a dictionary. After that is done, solving it is simply looping over the list to check which counts have been used and decrement the necessary parts of the dictionary. 


```python
S = 'example'

def solution(S):
    import collections
    counter = collections.Counter(S)
    unique_letters = len(counter)
    counter = dict(counter) #set counts of unique letters as a dictionary
    counter_values = list(counter.values()) #find unque counts

    num_changes = 0 #variable to record number of deletions
    values_seen = list(set(counter_values)) #get the unique counts in the original data 
    values_used = list(set(counter_values)) #track which of the counts have already been used

    for index, val in enumerate(counter_values): #for each unique counter value
        if val in values_seen: #if the value is still in the values_seen list, remove it to indicate that it has been "used"
            values_seen.remove(val)
        else: #otherwise, if value is not in the values_seen list, it means that it is not a valid value
            while (counter_values[index] in values_used) and (counter_values[index] != 0): #while the is found in the values_used list AND the value is non zero
                counter_values[index] -= 1 #subtract 1 from the counter value
                num_changes += 1 #add 1 to the change tracker
            values_used = list(set(counter_values)) #once we arrive at a value that is no longer in values_used, we recompute the set of values_used to add in the new number

    return(num_changes)

solution(S)
```
    4


