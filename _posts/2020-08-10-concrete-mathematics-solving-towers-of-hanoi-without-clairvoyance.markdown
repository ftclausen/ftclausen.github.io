---
title: "Concrete Mathematics: Solving Towers of Hanoi recurrence without clairvoyance"
date: 2020-08-10 12:25
classes: wide
use_math: true
categories: mathematics concrete-mathematics
---

This post expands, in terms of verbosity, what is presented in equation 1.3 where they say

> It doesn't take genius to discover that the solution to _this_ recurrence is just $U_n = 2^n$; hence $T_n = 2^n - 1$.
Even a computer could discover this.

This was not immediately obvious to me because a) I'm early in my maths adventures and b) I don't have the mathematical
fluency that comes with either working in the field or being in full-time study. What I needed to do was unroll that
recurrence both in symbolic and concrete forms to "get it". For reference, the recurrences are labelled as 1.1 in the
book, I'll put them here

$$
\begin{align}
T_0 &= 0; \\
Tn &= 2T_{n-1} + 1 \\
\end{align}
$$

# Setting the Scene

We are trying to find a solution to the recurrence $Tn = 2T_{n-1} + 1$. It helped me to expand this to bear a more direct
relevance to actually solving the Towers of Hanoi puzzle

$$
T_n = T_{n-1} + 1 + T_{n-1}
$$

To turn the above recurrence into a sentence: We first move all disks above the bottom most one to the middle peg
($T_{n-1}$), then move the bottom disk to the destination peg ($+1$), and finally the disks on the middle peg are moved
to the destination ($T_{n-1}$).

Now the authors suggest adjusting the recurrence by adding $1$ to both sides turning it into

$$
\begin{align}
T_0 + 1 &= 1; \\
T_n + 1 &= 2T_{n-1} + 2 \\
 &= T_{n-1} + 1 + T_{n-1} + 1 \\
\end{align}
$$

Notice we now have a common form we can reference easily: $T_n + 1$ where $n$ is just a placeholder. It can be $n-1$ as
well. As said on Math SE by Jaap Scherphuis [over
here](https://math.stackexchange.com/questions/2899754/concrete-mathematics-book-i-dont-understand-conversion-in-1-1-to-1-3#comment5989495_2899754).

# Simplifying

Now, with the "common form", $T_n + 1$, above we can assign that a simple label. The book uses $U_n$, as in $U_n = T_n +
1$. Now we can make our recurrence super simple to reason about

$$
\begin{align}
U_0 &= 1 \\
U_n &= 2U_{n-1} \\
\end{align}
$$

Reminder: $U_{n-1} = T_{n-1} + 1$. To see how we end up with the recurrence just being $U_n = 2^n$ we can unroll this
both symbolically and concretely

$$
\begin{align}
U_n &= 2U_{n-1} \\
    &= 2(2(U_{n-2})) \\
    &= 2(2(..(2U_{n-n}))) \\
\end{align}
$$

The $2^n$ nature of it is becoming more clear. As for a concrete example where $n=3$

$$
\begin{align}
U_3 &= 2(U_2) \\
     &= 2(2(U_1)) \\
     &= 2(2(2(U_0))) \\
     &= 2(2(2(1))) \\
     &= 8 \\
     &= 2^3 \\
\end{align}
$$

To get from that to our final closed form of $T_n = 2^n - 1$ we have, as discussed above

$$
U_n =  2^n \\
$$

But also

$$
\begin{align}
U_n &= T_n + 1 \\
\therefore \\
T_n + 1 &= 2^n \\
T_n &= 2^n - 1 \\
\end{align}
$$

> Even a computer could discover this.

This human was glad to work it out eventually.
