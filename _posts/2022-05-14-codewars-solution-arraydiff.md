---
title: "Codewars Solution Arraydiff"
date: 2022-05-14T10:20:49+02:00
categories: [programming, python]
tags: [python, codewars]
---

A solution to an Arraydif challenge on codewars (in python).

<!--more-->

## The Challenge

Your goal in this kata is to implement a difference function, which subtracts one list from another and returns the result.
It should remove all values from list a, which are present in list b keeping their order.
So array_diff([1,2,2,2,3],[2])
should leave this for list a: [1,3]

I came up with this:

```python
def array_diff(a, b):
    for item in b:
        try:
            while True:
                a.remove(item)
        except ValueError:
            pass
    print(a)
    return(a)
```

On a positive note all I can say is: it's A solution but not THE solution 😀
After checking the best practices,a **list comprehension** is so much better. You live, you learn:

```python
def array_diff(a, b):
    return [x for x in a if x not in b]
```

Which is a very elegant way of saying, give me everything in list a that is not in list b.
I read on a topic about writing code that this is mostly how things go. You first focus on solving the problem after which you end up with a version of working but inefficient code.  The next next step should be, to go through the code and refactor anything that improves readability and efficiency.
