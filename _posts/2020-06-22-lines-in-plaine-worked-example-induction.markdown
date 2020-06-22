---
title: "Concrete Mathematics Book: Worked example for \"Lines in the Plane\" Induction Step"
date: 2020-06-22 20:40
classes: wide
categories: mathematics
---

In the book [Concrete
Mathematics](https://www.amazon.com.au/Concrete-Mathematics-Foundation-Computer-Science/dp/0201558025) by Graham, Knuth,
and Patashnik there is a section in Chapter 1 called "Lines in the Plane". See the book for details but it is, basically
, a question of what is the max number of $L_n$ areas defined by $n$ lines through the plane they are on. So a plane
with no lines through it has 1 region, one line has 2 regions, three lines has 7 regions, etc.

# The Recurrence and Closed Form

The recurrence (labelled as 1.4 in the book) is

$$
\begin{align}
L_0 & = 1\\
L_n & = L_{n-1} + n \text{, for n > 0}
\end{align}
$$

And the proposed closed form (labelled as 1.6 in the book) we want to prove via induction is

$$
L_n = \frac{n(n+1)}{2} + 1 \text{, for n > 0}
$$

# The Book's Presentation of the Induction Step

The book says:

> The key induction step is:
>
$$
L_n = L_{n-1} + n = (\frac 12(n-1)n+1) + n = \frac 12n(n+1)+1
$$

I found this quite terse and not fully clear on first reading; I realised I needed more practice at induction. Once a
bit more [versed](/mathematics/induction/) in this topic I realised it was nice example of induction. I shall attempt to
explain my take on it here. Firstly the recurrence again:

$$
\begin{align}
L_0 & = 1\\
L_n & = L_{n-1} + n,\ \ for\ \ n > 0
\end{align}
$$

And then the proposed closed for again:

$$
L_n = \frac{n(n+1)}{2} + 1
$$

What we want is to "link" the two together. So we can take the "framework" of the recurrence above and "plug in" the
$L_{n-1}$ closed form. The $L_{n-1}$ closed form is, literally, subtracting one from the $L_n$ closed for terms:

$$
L_{n-1} = \frac{(n-1)n}{2} + 1
$$

So, once more with gusto and blue highlighting to show the "plugging in", the recurrence is

$$
L_n = \color{blue}{L_{n-1}} + n,\ \ for\ \ n > 0
$$

Now we plug in the $L_{n-1}$ version of the closed form

$$
L_n =  \color{blue}{\frac{(n-1)n}{2} + 1} + n
$$

Then (with Algebra!) we can prove the induction

$$
\begin{align}
L_n & =  \color{blue}{\frac{(n-1)n}{2} + 1} + n \\
& = \frac{(n-1)n}{2} + \frac 22 + \frac{2n}{2} \\
& = \frac{n^2 - n + 2 + 2n}{2} \\
& = \frac{n^2 + n + 2}{2} \\
& = \frac{n(n + 1) + 2}{2} \\
& = \frac{n(n + 1)}{2} + \frac 22 \\
& = \frac{n(n + 1)}{2} + 1
\end{align}
$$

And so we get our closed form back meaning the induction is complete.
