---
title: "Sums of sums visually and as nested \"for\" loops"
date: 2020-11-14 20:05
classes: wide
use_math: true
categories: mathematics dev
---

In Concrete Mathematics (Graham, Patashnik, Knuth) I am working my way through page 36 about sums of sums. Specifically
the "rocky road" (yum)  method for swapping the order of summation when the range of the inner sum depends on the index
variable on the outer sum.

I find the Iversonian notation confusing to start with and now I see it's "just symbols" and that, for example, $K'$
does not mean the complement of some set $K$ _in that context_. The ever helpful Brian M. Scott helper [clear that up
for
me](https://math.stackexchange.com/questions/3897525/concrete-mathmatics-notation-for-sums-of-sums-where-inner-sum-depends-on-outer):

> The sets $K'$ and $J'(k)$ for $k \in K'$ are not defined in (2.30): (2.30) simply describes in very general terms a kind
> of rearrangement of a double summation — reversing the order of summation — that is often very helpful, just as
> reversing the order of integration in a double iterated integral is often very helpful.

To help cement my knowledge I've decided to explicitly visualise and also bring sums of sums to something I do know:
nested loops. Standard disclaimer of "maybe I have this wrong" applies.

I'll get into the Iversonian in the last section but, for now, here is a reversed summation I'll use as a reference for
below

$$
\sum_{j=1}^n\sum_{k=j}^na_{j,k}=\sum_{1\le j\le k\le n}a_{j,k}=\sum_{k=1}^n\sum_{j=1}^ka_{j,k}\,,
$$

Specially the RHS and LHS.

# Visual Overview

Basically by reversing the order of summation we are going from counting by rows first to counting by columns first:

![images/cm_2.32.png](/images/cm_2.32.png)

Pretty simple really

# As Nested Loops

NOTE: This illustration is focused on the _index variables_ not the actual summation operation. We're iterating over an
existing table to show how the results end up the same when swapping the indexes. In a "real" summation we'd be summing
over the previous value not some pre-existing data set.

For loops, especially classic ones, are basically summations in code. This Python snippet adds up an array of arrays
"table" where the inner loop depends on the index of the outer loop

```python
#!/usr/bin/env python3

# Random table of things to add up - we are adding the upper right triangle. So
#  [ 19555, 2038,  3413  ],
#  [        35620, 3734  ],
#  [               34234 ]
table = [
  [ 19555, 2038, 3413  ],
  [ 3432, 35620, 3734  ],
  [ 4543, 43543, 34234 ]
]

n = len(table)

# loop1
total1 = 0
for j in range(0, n):
    for k in range(j, n):
        total1 += table[j][k]

# loop2
total2 = 0
for k in range(0, n):
    # We need k+1 because range is not inclusive of last number
    for j in range(0, k+1):
        total2 += table[j][k]

print(f'Total1 = {total1}')
print(f'Total2 = {total2}')
```

Where `loop1` is 

$$
\sum_{j=1}^n\sum_{k=j}^na_{j,k}
$$

And `loop2` is

$$
\sum_{k=1}^n\sum_{j=1}^ka_{j,k}
$$

When it runs it prints

```
$ ./nested_loops.py
Total1 = 98594
Total2 = 98594
```

Showing the two summation approaches are adding up the same section because it is not a symmetric table.

# More Generally

The book uses a general notation that, as mentioned in the intro above, confused me. This section is more a brain dump
of me attempting to understand this and I'll tweak it as I gain more understanding. They present this equation

$$
\sum_{j \in J} \sum_{k \in K(j)} a_{j,k} = \sum_{k \in K'} \sum_{j \in J'(k)}a_{j,k}
$$

And say

> Here the sets $J$, $K(j)$, $K'$, and $J'(k)$ must be related in such a way that
>

$$
[j \in J][k \in K(j)] = [k \in K'][j \in J'(k)]
$$

These are just symbols and, for example, $K'$ is not meaning the complement of some set. It's just showing to "return 1"
when the inner sum's condition is met. Dissecting even more the conditions are

1. $[j \in J]$ and $[k \in K']$ where the book says "let $J = K'$ be the set of all integers. So this will return 1 all
   the time.
2. $[k \in K(j)]$ and $[j \in J'(k)]$ is the outer sum dependent property. So, for example, "loop from current value k" or
 "loop to current value of k". The book calls this "the basic property $P(j,k)$ that governs a double sum". This will
 return 0 _except_ when we are in the "upper right triangle".

Furthermore #1 above is the main loop and #2 is the nested loop in the code example.

