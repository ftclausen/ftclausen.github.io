---
title: "Really Simple Mathematical Induction Example"
date: 2020-05-03 21:00
classes: wide
categories: mathematics
---

I am going to attempt to explain my understanding of mathematical induction (the weak induction variety) using a super simple
example. This is as much to cement my knowledge as to potentially help others. 

# Setting the Scene

I'm selecting a somewhat random recurrence taken from an exercise on Khan Academy's [arithmetic series
exercise](https://www.khanacademy.org/math/old-integral-calculus/series-ic/series-tut-ic/e/arithmetic_series?modal=1).
So this does not represent anything real and the base case was randomly generated.

The exercise uses the following recurrence and asks you to find the sum of 880 terms. 

$$
a_1 = -53 \\
a_i = a_{i-1} + 6
$$

So you have to find a "closed form" so that you don't have to actually do 880 iterations (or write a program to do it).
By writing out the first few iterations you get

$$
a_1 = -53 \\
a_2 = -47 \\
a_3 = -41 \\
a_4 = -35 \\
$$

Thus it would appear our closed form is

$$
a_i = 6(i - 1) - 53
$$

Of course using induction for this simple example might seem excessive but it helps me understand how induction works.

# The Induction Part

What we want to prove is that the current iteration is the same as the previous iteration plus the increment (6 in this
case). In other words that $a_i = a_{i-1} + 6$. This is the bit I initially found confusing; we are going to use the
scaffolding of the recurrent form $a_{i-1} + 6$ to plug in our closed form.

So we have two $a_i$'s, the recurrent form:

$$
a_i = a_{i-1} + 6
$$

and the closed form:

$$
a_i = 6(i-1) - 53
$$

So if we make our close form the LHS and our recurrence _with the closed form plugged in_ our HHS we get (with the first
line being a reminder of above)

$$
\begin{align}
a_i\ \ \ \ \ & = \color{blue}{a_{i-1}} + 6 \\
6(i-1) - 53 & = \color{blue}{6(i-2) - 53} + 6
\end{align}
$$

Highlighted in <span style="color:blue">blue</span> is where we substituted in our closed form, tweaked to get the
previous iteration, into the recurrence "scaffolding". As mentioned, we had to tweak our closed form to get the "minus
one" or previous iteration.  Hence it is $6(i-2) - 53$; subtracting two instead of one. This is so we can plug it into
the recurrence form "scaffold" which expects $a_{i-1}$.

Thus, if the LHS = RHS we have proven the induction. Doing some algebra we can get there

$$
\begin{align}
6(i-1) - 53 & = 6(i-2) - 53 + 6 \\
6i -6 -53 & = 6i -12 - 53 + 6 \\
6i - 59 & = 6i - 59 \\
\end{align}
$$

Thus LHS = RHS and the closed form is proved. QED as they say.

PS: For those still following along the sum of 880 terms was 2273920.
