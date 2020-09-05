---
title: "Concrete Mathematics: Quick geometric series with Towers of Hanoi Clarification"
date: 2020-09-05 21:25
classes: wide
use_math: true
categories: mathematics concrete-mathematics
---

This concerns the paragraph just before equation 2.9. I will quote it here

> The sum of the geometric series $2^{-1} + 2^{-2}+ \cdot\cdot\cdot + 2^{-n} = (\frac{1}{2})^1 + (\frac{1}{2})^2 +
> \cdot\cdot\cdot + (\frac{1}{2})^n$ will be derived later in this chapter; it turns out to be $1 - (\frac{1}{2})^n$.
> Hence $T_n = 2^nS_n = 2^n - 1$

That last bit

$$
T_n = 2^nS_n = 2^n - 1
$$

Had me confused for a while but, after stopping overthinking it, I came to release they mean things quite literally
using the previous definitions in the book with some algebraic manipulations

$$
\begin{align}
\frac{T_n}{2^n} &= S_n \\
T_n &= 2^nS_n \\
\end{align}
$$

Now at this point we can substitute in the definition for $S_n$ which is the sum of the geometric series given to us in
the quote above as $1 - (\frac{1}{2})^n$. First we slightly rearrange it as $1 - \frac{1}{2^n}$ then plug that in

$$
\begin{align}
T_n &= 2^n(1 - \frac{1}{2^n}) \\
T_n &= 2^n - \frac{2^n}{2^n} \\
T_n &= 2^n - 1
\end{align}
$$

Tada! We get the closed form for the Towers of Hanoi recurrence.
