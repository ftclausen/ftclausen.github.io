---
title: "Josephus Problem Revisited"
date: 2022-02-12 14:09
classes: wide
use_math: true
categories: mathematics
---

I am restarting Concrete Mathematics (Graham, Knuth, Patashnik) due to a forced break, neurological
problem, will spare details. Maths is my friend in the recovery process even
though it is challenging. Perhaps because it is challenging.

As with my other posts this will expand, more verbosely, on what was presented
in Concrete Mathematics.

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

$J(n)$ seems to be always odd even if you start _input to $J(n)_ with even numbers. I'll park the
odd case _input_ for now and cover in a separate sub-section. For me, the best
way I can understand this is to say the recurrence as a sentence (from the book) and then show
as an algebraic expression.

*SENTENCE*

> The _output_ of $J(n)$ is always odd. There is a good reason for this:  The
first trip around the circle eliminates all the even numbers. Furthermore, if
$n$ itself is an even number, we arrive at a situation similar to what we began
with, except that there are only half as many people, and their numbers have
changed.

The description above kind of inverts what the recurrence equation says. In
words:

- _With words_: There are only half as many people and their numbers have changed
- _Recurrence itself_: There will be twice as many people and their numbers will 
    change

So if we have $2n$ people originally then we'll have $n$ after the first round.
I.e. if we want to fine the next rung of the recurrence then we need to "ask"
for $2n$ not $n + 1$. So this recurrence is exponential. We'll find the answer
rather quickly. I *think* this is the left-hand-side of the recurrence:

$$
\color{blue}J(2n)\color{#3d4144} = 2J(n) - 1
$$

(Because anything $2n$ is going to be even)

Then,

> This is just like starting out with $n$ people, except that each person's
number has been doubled and decreased by one.

I _think_ the above sentence is the right-hand-side of the recurrence:

$$
J(2n) = \color{blue}2J(n) - 1
$$

Hence this is how the recurrence is constructed. I hope to develop this kind of
methodical thinking via this book so that, I too, can make such derivations of
my own one day.

## Odd case

> Odd case? Hey, leave my brother out of it

-- The graffiti on page 10

As the book says

> With $2n + 1$ people, it turns out that person number 1 is wiped out just
after person number 2n.

(Because anything $2n + 1$ is going to be odd)

See the book for a beautiful Latex (or maybe Tex) diagram but below is, with
more verbosity, what the diagram says (below for n = 5):

- $2n + 1$ is $9$
- $2n - 1$ is $7$
- $2n - 3$ is $5$
- $2n - 5$ is $3$
- $2n - 7$ is $1$

I think this is the LHS of recurrence because anything $2n + 1$ is odd:

$$
\color{blue}J(2n + 1)\color{#3d4144} = 2J(n) + 1
$$

Then,

> Again we have almost the original situation with $n$ people, but this time
their numbers are doubled and _increased_ by $1$.

I think this is the RHS of the recurrence because anything $2J(n) + 1$ is going
to be odd:

$$
J(2n + 1) = \color{blue}2J(n) + 1
$$

## Putting it all together

With the base case it all ends up looking like

$$
\begin{align}
J(1) &= 1 \\
J(2n) &= 2J(n) - 1, \ \ \ \ \ \ \text{for} \ n \ge 1; \\
J(2n + 1) &= 2J(n) + 1, \ \ \ \ \ \ \text{for}\ n \ge 1; \\
\end{align}
$$

NEXT: Step through recurrence manually
NEXT: Look at closed forms if necessary (check previous posts)
