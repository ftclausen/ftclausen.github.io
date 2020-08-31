---
title: "Concrete Mathematics: Repertoire Method Section 2.2 - The Sequel"
date: 2020-08-29 21:00
classes: wide
use_math: true
categories: mathematics concrete-mathematics
---

As with most of my posts on Concrete Mathematics this is a more verbose outline of the repertoire method in section
2.2. It explains how to find the closed form to the sum recurrence 2.7 which I reproduce here

$$
\begin{align}
R_0 &= \alpha \\
R_n &= R_{n-1} + \beta + \gamma n
\end{align}
$$

Then, as in chapter one, we work through a few small cases like $R_1 = \alpha + \beta + \gamma$, $R_2 = \alpha + 2\beta + 3\gamma$ and so on.
We can see that there is $\alpha$, $\beta$, $\gamma$ (the "general parameters") which have "something done" to them to give us the final closed
form. This "something we do" is represented by, at this point, unknown functions $A(n)$, $B(n)$, and $C(n)$ (a.k.a.
coefficients of dependence on the before mentioned general parameters). Putting them together we have a framework for starting
to find our closed form

$$
R_n = A(n)\alpha + B(n)\beta + C(n)\gamma
$$

What the book does is cleverly plug in "simple functions of n for $R_n$" and letting the right-hand-side shake itself out. With some
luck we'll just have one of $\alpha$, $\beta$, or $\gamma$ left equalling the "simple function of n". Let's put it to
action

# Finding $A(n)$

Here they set $R_n = 1$ and, lo and behold, the general parameters end up as $\alpha = 1$, $\beta = 0$, $\gamma = 0$.
Now, why this is confused me (I could still be confused but not know it), until I pondered the logical implications of
setting $R_n = 1$. **NOTE** it is not the same as setting $n = 1$; for this repertoire item _the whole sum_ is 1

So if the whole sum is $1$ what does that mean? Using the recurrence it must mean that $S_{n-1}$ is $1$, $\beta$ is $0$,
and $\gamma$ is $0$. If $\beta$ and $\gamma$ were anything other than zero then the sum would not be $1$. Hence $\alpha
= 0$

Now, using that info, we can sub-in $\alpha = 1$, $\beta = 0$, $\gamma = 0$ to the closed form

$$
\begin{align}
1 &= A(n)\cdot 1 + B(n)0 + C(n)0 \\
1 &= A(n)
\end{align}
$$

# Finding $B(n)$

With similar logic to above, setting $R_n = n$, it means _the whole sum_ is the same as $n$. The only way that could be is
if we're simply adding $1$ with each recurrence iteration. So it would be like a recurrence of $R_n = S_{n-1} + 1 +
0(n)$ meaning $\alpha = 0$ (the first iteration would be 0) and $\gamma = 0$ otherwise we'd have a result of $R_n > n$.
Hence $\alpha = 0$, $\beta = 1$, $\gamma = 0$. Plugging in those values to our proposed closed form we get

$$
\begin{align}
n &= A(n)\cdot0 + B(n)\cdot1 + C(n)\cdot0 \\
n &= B(n)
\end{align}
$$

# Finding $C(n)$

This one I didn't figure out without some help from Professor Internet in the form of Hans Lundmark in [a
comment](https://math.stackexchange.com/questions/67939/repertoire-method-clarification-required-concrete-mathematics#comment161060_67946).
For this particular repertoire item the authors are setting $R_n$ to $n^2$. With that done it implies $\alpha = 0$ because $0^2 = \alpha$. To then see what $\beta$ and $\gamma$ need to be in order to eliminate them we simply plug in
$n^2$ to the recurrence RHS so

$$
\begin{align}
n^2 &= (n - 1)^2 + \beta + \gamma n \\
n^2 &= n^2 -2n +1 + \beta + \gamma n
\end{align}
$$

At this point we do some selective factoring in order to see what $\beta$ and $\gamma$ need to be set to
in order to get our simple repertoire item for $C(n)$. First re-arrange to make it more obvious (swap $\beta$ and
$\gamma$)

$$
\begin{align}
n^2 &= n^2 - 2n + \gamma n + \beta \\
n^2 &= n^2 + n(\gamma -2) + 1(1 + \beta)
\end{align}
$$

And, tada, we can see we need to set $\gamma = 2$ and $\beta$ to $-1$. Using this info we can plug it back into our
proposed closed form:

$$
\begin{align}
n^2 &= A(n)\cdot0 + B(n)\cdot-1 + C(n)\cdot2 \\
n^2 &= 0 -B(n) + 2C(n) \\
n^2 &= 2C(n) -B(n)
\end{align}
$$

At this point we know $B(n) = n$ (from previous repertoire step for $B(n)$) so sub that in and rearrange to get the result

$$
\begin{align}
n^2 &= 2C(n) - n \\
n^2 + n &= 2C(n) \\
\frac{n^2 + n}{2} &= C(n)
\end{align}
$$
