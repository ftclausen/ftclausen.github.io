---
title: "Concrete Mathematics: Notes on Josephus Problem Induction"
date: 2022-04-07 12:00
classes: wide
categories: mathematics concrete-mathematics
use_math: true
---

The explanations in Concrete Mathematics are very good if a bit terse. So, [as
before](../mathematics/lines-in-plaine-worked-example-induction/), I will add some extra detail in a blog post to help
cement (🥁) my knowledge. I am also shouting it into the void in case it helps someone else.

See the book for full context; I'll just cover the induction related steps.

# The Recurrences and Closed Form

The recurrence comes in two flavours: odd and even. The complete recurrences are thus:

$$
\begin{align}
J(1) &= 1;\\
J(2n) &= 2J(n) - 1, \ \ \ \ \text{for n} \geq 1\\
J(2n+1) &= 2J(n) + 1, \ \ \ \ \text{for n} \geq 1\\
\end{align}
$$

And the closed form is as follows

$$
J(2^m+\ell)=2\ell+1
$$

# Even Induction

The book goes on to give an example of the even induction

> $J(2^m+\ell)=2J(2^{m-1} + \ell/2) - 1 = 2(2\ell/2 + 1) - 1 = 2\ell + 1$
>
> by (1.8) and the induction hypothesis; this is exactly what we want.

What confused me at first was the leap

$$ 2J(2^{m-1} + \ell/2) - 1 = 2(2\ell/2 + 1) - 1 $$

This is the key induction step - this is not done via algebraic manipulation. We are substituting the closed form that
is equivalent to $J(n)$. Our closed form proposal is actually for $2J(n)-1=2l+1$ so we need to refit it to be for $J(n)$. Thus
$l$ is in the previous power of two block (remember $J(n)$ not $2J(n)$) and half as big: $l/2$. So the $J(n)$ closed form is $2l/2 +1$.

So we plug that in and then he next step, however, _is done_ via algebra:

$$ 2(2\ell/2 + 1) - 1 = 2\ell + 1 $$

Completing the even induction.

# Odd Induction

Credit: I was stuck on this until helped out by the ever helpful Brian M. Scott on [Math
Stackexchange](https://math.stackexchange.com/a/3743359/109665). I will repeat
that here and then do my own (simplified) take on thereafter. First Brian's
reply:

> Your're not applying the recurrence for the odd case correctly. Suppose that
$2n + 1 = 2^m + \ell$, where $0 \leq \ell \leq 2^m$. The recurrence is $J(2n +
1) = 2J(n) + 1$, and _n_ here is $\frac12(2^m + \ell - 1)$, so
>
$$
\begin{align}
J(2^m + \ell) &= 2J\left(2^{m-1} + \frac{\ell - 1}{2}\right) + 1 \\
              &= 2\left(\frac{2(\ell - 1)}{2} + 1 \right) + 1 \\
              &= 2\ell + 1
\end{align}
$$
>
> as desired

I will do some colour highlighting and ever more verbose explanation for my
future self here. The core is we want to convert $2n + 1$ into $n$ hence
dividing by two and subtracting one:

$$
\frac{2(n + 1)}{2} - 1 = n
$$

We apply these operations during the induction step to turn $2\ell + 1$ into
$\frac{2(\ell -1)}{2} + 1$. Notice the division by two and subtraction by one?
And now with gusto and in context of the induction step where we "insert" the
closed form representing _n_ into the recurrence "framework" (substitution
highlighted in blue):

$$
\begin{align}
J(2^m  + \ell) &= 2\left(\color{blue}\frac{2(\ell - 1)}{2} + 1\color{#3d4144}\right) + 1 \\
               &= 2\ell + 1
\end{align}
$$

Giving us what we want.