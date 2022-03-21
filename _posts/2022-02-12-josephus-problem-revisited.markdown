---
title: "Josephus Problem - Recurrence - Revisited - Unrolled"
date: 2022-02-12 14:09
classes: wide
use_math: true
categories: mathematics
---

I am restarting Concrete Mathematics (Graham, Knuth, Patashnik) due to a forced
break due to a neurological problem called [Anti-NMDA receptor
encephalitis](https://en.wikipedia.org/wiki/Anti-NMDA_receptor_encephalitis). I
include this detail because it is a relatively newly discovered disease and
misdiagnosis is common so, in a very small way, raising awareness.

Maths is my friend in the recovery process even though it is challenging. Perhaps because it is challenging.

This post goes over the _recurrence only_ because I found that challenging. For
some reason I find recursion harder than it should be 😅

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
* ... and so on - see above linked
    [video](https://www.youtube.com/watch?v=uCsD3ZGzMgE) for a great
    explanation.

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

I think the point the book is trying to make is that, at each go around, we are
essentially at the same point as the round before except there are _half_ as
many people _and their numbers (aka "label" or "position") have changed. Or, [as
Wikipedia puts it](https://en.wikipedia.org/wiki/Josephus_problem):

> The first time around the circle, all of the even-numbered people die. The
> second time around the circle, the new 2nd person dies, then the new 4th
> person, etc.; it is as though there were no first time around the circle.

# The Recurrence

This refers heavily to [this
post](https://math.stackexchange.com/a/4069794/109665) by, the ever helpful,
Brian M. Scott. I am only adding even more details for my addled brain to help
it understand.

First the recurrences (for base case, odd, and even)  which is the notational
expression of the above sentences:

$$
\begin{align}
J(1) &= 1 \\
J(2n) &= 2J(n) - 1, \ \ \ \ \ \ \text{for} \ n \ge 1; \\
J(2n + 1) &= 2J(n) + 1, \ \ \ \ \ \ \text{for}\ n \ge 1; \\
\end{align}
$$

## Unrolling

The [post](https://math.stackexchange.com/a/4069794/109665) by Brian M. Scott
shows the case for $n = 21$ but I found it useful to step through the small
cases as well. 

Formatting note: I am using a comma followed by some notes. The font for the
`\text{...}` is not beautiful but serves the purpose.

###  n = 1

This is the base case so $J(1) = 1$

### n = 2

$$
\begin{align}
J(2) &= J(1\cdot2), \quad \text{expand} \\
     &= 2J(1) - 1, \quad \text{recurse} \\
     &= 2\cdot1 - 1 = 1, \quad \text{base case and finish}
\end{align}
$$

### n = 3

$$
\begin{align}
J(3) &= J(1\cdot2 + 1), \quad \text{expand} \\
     &= 2J(1) + 1, \quad \text{recurse}\\
     &= 2\cdot1 + 1 = 1, \quad \text{base case and finish}\\
\end{align}
$$

### n = 4

$$
\begin{align}
J(4) &= J(2\cdot2), \quad \text{expand}\\
     &= 2J(2) - 1, \quad \text{recurse} \\
     &= 2J(1\cdot2) - 1, \quad \text{expand} \\
     &= 2(2J(1) - 1) - 1, \quad \text{recurse} \\
     &= 4J(1) - 3, \quad \text{simplify}\\ 
     &= 4\cdot1 - 3 = 1, \quad \text{base case and finish} 
\end{align}
$$

### n = 5

$$
\begin{align}
J(5) &= J(2\cdot2 + 1), \quad \text{expand} \\
     &= 2J(2) + 1, \quad \text{recurse} \\
     &= 2J(1.2) + 1, \quad \text{expand} \\
     &= 2(2J(1) - 1 ) + 1, \quad \text{recurse} \\
     &= 4J(1) - 2 + 1, \quad \text{simplify} \\
     &= 4\cdot1 - 1 = 3, \quad \text{base case and finish}
\end{align}
$$

### n = 6

$$
\begin{align}
J(6) &= J(2\cdot3), \quad \text{expand} \\
     &= 2J(3) - 1, \quad \text{recurse} \\
     &= 2J(1\cdot3) - 1, \quad \text{expand} \\
     &= 2(2J(1) + 1) - 1, \quad \text{recurse} \\
     &= 2(2\cdot1 + 1) - 1 = 4 + 2 - 1 = 5, \quad \text{base case and finish}
\end{align}
$$

# Conclusion

Doing this manual unrolling really helps understand the nature of the problem
when expressed in math notation. We can see how powers of two play a role which
is useful in the closed for solution(s).
