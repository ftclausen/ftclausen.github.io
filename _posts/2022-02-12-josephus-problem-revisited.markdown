---
title: "Josephus Problem Revisited"
date: 2022-02-12 14:09
classes: wide
use_math: true
categories: mathematics
---

I am restarting Concrete Mathematics (DEK) due to a forced break, neurological
problem, will spare details. Maths is my friend in the recovery process even
though it is challenging. Perhaps because it is challenging.

As with my other posts this will expand, more verbosely, on what was presented
by DEK in 1.3.

# Setting the Scene with Some Examples

While the concept of shooting each _second_ remaining person sounds obvious I
found it useful to step through it on pen and paper which I will attempt to
reproduce here in text; really an animation is also useful to look at such as
[this one](https://www.youtube.com/watch?v=uCsD3ZGzMgE). I won't show a diagram
because all you see is the end result which is not so useful

* $n = 1$ - Survivor is $1$ obviously because Josephus does not want to kill
     himself.
* $n = 2$ 
  * 1 shoots 2
  * 1 survives
* $n = 3$
  * 1 shoots 2
  * 3 shoots 1
  * 3 survives
* $n = 4$
  * 1 shoots 2
  * 3 shoots 4
  * 1 shoots 3
  * 1 survives
* $n = 5$
  * 1 shoots 2
  * 3 shoots 4
  * 5 shoots 1
  * 3 shoots 5
  * 3 servives
* $n = 6$
  * 1 shoots 2
  * 3 shoots 4
  * 5 shoots 6
  * 1 shoots 3
  * 5 shoots 1
  * 5 survives

I was accidentally thinking the dead person was also shooting someone (😅) which
meant I was messing up above. Here is a tabular form of the above as presented in
the book:

$$
\begin{array}{|c|c|}
\hline
n & 1\ \ 2\ \ 3\  \ 4\ \ 5\ \ 6\
\\
\hline
J(n) & 1\ \ 1\ \ 3\ \ 1\ \ 3\ \ 5\
\\
\hline
\end{array}
$$

# The Quest for a Recurrence

This part I find tricky; both in words and notation.

## Even Case

$J(n) seems to be always odd even if you start with even numbers. I'll park the
even case for now and cover in a separate sub-section.


