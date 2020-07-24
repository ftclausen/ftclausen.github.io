---
title: "Concrete Mathematics: Repertoire Method Notes"
date: 2020-07-24 21:42
classes: wide
categories: mathematics
use_math: true
---

As always, these are my notes to help me cement my knowledge after reading sections of Concrete Mathematics I find
particularly challenging or interesting. Usually both. This section uses an example of the _repertoire method_ for
finding closed forms to recurrences; basically breaking the problem down into smaller sub-problems that can hopefully be
more easily solved. Then plugging in these simpler solutions to solve for the later problem. Plug and chug.

# Generalised Josephus Recurrence

See the book for full context but basically we want to find the closed form for the following generalised Josephus
recurrences:

$$
\begin{align}
f(1) &= \alpha \\
f(2n) &= 2f(n)+\beta ,\ \ \ \ \ \text{for n} \geq 1 \\
f(2n+1) &= 2f(n)+\gamma ,\ \ \ \ \ \text{for n} \geq 1 \\
\end{align}
$$

In the book they then step through some small values, starting with $f(1)=\alpha$ and then working their way up. I
[needed a nudge](https://math.stackexchange.com/a/3507144/109665) before I fully grokked how they were using these
recurrences to construct the below table

$$
\begin{array}{|c|lc|}
\hline
n& f(n)
\\ \hline
1 &\ \ \alpha
\\ \hline
2 & 2\alpha+\beta
\\ 3 & 2\alpha\ \ \ \ \ \ \ \ \ \ \ \ +\ \ \gamma\\
\hline
4 & 4\alpha + 3\beta\\
5 & 4\alpha + 2\beta\ \ \ +\ \ \gamma\\
6 & 4\alpha + \ \ \beta\ \ \ +2\gamma\\
7 & 4\alpha + \ \ \ \ \ \ \ +3\gamma\\
\hline
8 & 8\alpha + 7\beta\\
9 & 8\alpha + 6\beta\ \ \ \ + \gamma
\\ \hline
\end{array}
$$

By pondering the above table for a sufficient amount of time, possibly with a caffeinated beverage, one can make the
following observations

- $\alpha$'s coefficient is $n$'s largest power of 2
- Between powers of two, $\beta$'s coefficient decreases by $1$ down to $0$
- While $\gamma$'s increases by $1$ up from $0$

So, to find some "scaffolding" with which we can express what the closed $f(n)$ might look like, we can create some
functions that apply to each of the constants

$$
f(n)=A(n)\alpha + B(n)\beta + C(n)\gamma
$$

So based on the above observations points we can figure out that each of those functions might do the following to the
constants they are coefficients of:

$$
\begin{align}
A(n) &= 2^m \\
B(n) &= 2^m - 1 - \ell \\
C(n) &= \ell \\
\end{align}
$$

With the constraints of $n = 2^m + \ell$ and $0 \leq l \lt 2^m,\ \text{for}\ n \geq 1$. So our proposed general closed
form could be

$$
f(n) = 2^m\alpha + (2^m - 1 - \ell)\beta + \ell\gamma
$$

The book says it's not "terribly hard" to prove the closed form by induction but that it's "messy and uninformative".
Maybe I'll try it some day but, for now, let's go over their repertoire method. This works by finding a series of "leg
ups" sub-problems to find the, currently unproven, $A(n)$, $B(n)$, and $C(n)$ functions.

The book goes about setting the constants to convenient values in order to eliminate the other unknowns as required. But
it's not quite like a set of simultaneous equations. We're actively looking for special cases we can use to eliminate
unknowns or find equivalences to something known.

Aside: How exactly the authors arrived at this application of the repertoire method is not explained; probably through
experience and intuition. Truly something worthwhile to develop in myself.

# Repertoire Item 1: Found by setting $\alpha=1, \beta=0, \alpha=0$

Setting $\beta$ and $\gamma$ to above special case values will eliminate them leaving behind just the $A(n)$
recurrences without the addition of the constants. Thus setting $\alpha = 1$, $\beta = 0$, and $\gamma = 0$ we will get

$$
\begin{align}
A(1) &= 1; \\
\color{blue}{A(2n)} &= 2A(n) + \beta =  2A(n) + 0 = \color{blue}{2A(n)},\ \ \ \ \text{for}\ n \geq 1; \\
\color{blue}{A(2n+1)} &= 2A(n) + \gamma = 2A(n) + 0 = \color{blue}{2A(n)},\ \ \ \ \text{for}\ n \geq 1; \\
\end{align}
$$

Since the RHS is $2A(n)$ we will be multiplying the preview power of two by two. Thus it will end up being $2^m$.

As for the LHS, it is the same as our specific closed for thus $A(2^m + \ell)$ hence the first part of our repertoire is

$$
A(2^m + l) = 2^m
$$

The book sums all this up as

> Sure enough, it's true (by induction on $m$) that $A(2^m + \ell) = 2^m$

The induction they talk about seems to be

$$
A(2^m + \ell) = 2A(n) = 2(2^{m-1}) = 2^m
$$

So, finally, this repertoire item ends up being

$$
A(n) = 2^m
$$

Note: $n$ can also be expressed as $2^m+\ell$ as above. It's still $n$ just decomposed (and, _I think_, in this
special case repertoire item $\ell$ is always $0$).

# Repertoire Item 2: Found by setting constant function $f(n)=1$

Setting $f(n)=1$ allows us to arrange the whole proposed closed form equal to something; this becomes important later on
when we're solving for the various values of $A(n)$, $B(n)$, $C(n)$. The book describes it as

> Next, let's use recurrence (1.11) and solution (1.13) _in reverse_, by starting with a simple function $f(n)$ and
seeing if there are constants ($\alpha$, $\beta$, $\gamma$) that will define it.

By "will define it" I think the book means will remove the generalised $\alpha$, $\beta$, and $\gamma$ and replace it
with an actual number. The way they do this is by defining $f(n)$ to a constant function $f(n)=1$ and plugging it into
the recurrences

$$
\begin{align}
1 &=\alpha; \\
1 &= 2\cdot1 + \beta; \\
1 &= 2\cdot1 + \gamma; \\
\end{align}
$$

Solving this shows us $(\alpha, \beta, \gamma) = (1, -1, -1)$. We can then replace $\alpha$, $\beta$, and $\gamma$ in
the proposed closed form which will result in our next repertoire item of

$$
A(n) - B(b) - C(n) = f(n) = 1
$$

# Repertoire Item 3: Found by setting constant function $f(n)=n$

The last item in our repertoire is obtained by setting another constant function, $f(n)=n$. This gives us

$$
\begin{align}
1 &= \alpha; \\
2n &= 2\cdot n + \beta; \\
2n+1 &= 2\cdot n + \gamma; \\
\end{align}
$$

Now we need to find when these equations are true with regards to the values of $\alpha$, $\beta$, and $\gamma$. It
turns out that this is when $\alpha=\color{blue}{1}, \beta=\color{blue}{0}, \gamma=\color{blue}{1}$. Explicitly:

$$
\begin{align}
1 &= \color{blue}{1}; \ \ \ \text{for}\ \alpha \\
2n &= 2\cdot n + \color{blue}{0}; \ \ \ \text{for}\ \beta \\
2n+1 &= 2\cdot n + \color{blue}{1}; \ \ \ \text{for}\ \gamma \\
\end{align}
$$

This then, when we plug the chosen values for $\alpha$, $\beta$, and $\gamma$ into our proposed closed form gives us

$$
\begin{align}
A(n)\alpha + B(n)\beta + C(n)\gamma &= n \\
A(n)\cdot 1 + B(n) \cdot 0 + C(n) \cdot 1 &= n \\
A(n) + C(n) &= n \\
\end{align}
$$

Ingeniously giving us a useful repertoire item because we can now find C(n) easily.

# That equation really tied the room together

Now we have a useful repertoire, one item per "independent parameter" i.e. $\alpha$, $\beta$, and $\gamma$:

$$
\begin{align}
A(n) &= 2^m \\
A(n) - B(n) - C(n) &= 1 \\
A(n) + C(n) &= n
\end{align}
$$

We can now set about solving for each of the hitherto mysterious $B(n)$ and $C(n)$ functions.

## Find $C(n)

Starting with $C(n)$

$$
A(n) + C(n) = n \\
C(n) = n - A(n) \\
C(n) = \ell \\
$$

Wait, what, $C(n) = \ell$? That's because of the following:

- $n = 2^m + \ell$ - From above repertoire item 1
- $A(n) = 2^m$ - Also from repertoire item 1

Thus

$$
C(n) = n - A(n) = 2^m + \ell - 2^m = \ell \\
\therefore
C(n) = \ell
$$

Nice.

## Find B(n)

Now that we have $C(n)$ we can line everything up to find $B(n)$

$$
\begin{align}
A(n) - B(n) - C(n) &= 1 \\
2^m - B(n) - \ell &= 1 \\
-B(n) &= -2^m + \ell + 1 \\
B(n) &= 2^m - \ell - 1 \\
\end{align}
$$

## Finally tie it together

Using the above repertoire method we arrive at the following function values:

- $A(n) = 2^m$
- $B(n) = 2^m - \ell - 1$
- $C(n) = \ell$

And, behold!, they match our initial observations proving they are correct.
