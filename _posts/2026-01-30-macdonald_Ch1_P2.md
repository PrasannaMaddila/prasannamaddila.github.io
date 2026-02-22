---
layout: post
title: "[Comm.Alg] Exercise on Polynomial Rings"
categories: math
---

I'm just dropping off for a small exercise in Atiyah and Macdonald's book on Commutative Algebra. Specifically, this is Chapter 1's Exercise 2 on Polynomial Rings. If you need a quick [Primer on Rings]({% post_url 2023-12-25-primer-on-rings %}), I've got a terse post for you.

# Some Definitions 

<div class="defn" text="Units, Zero-Divisors and Nilpotents" markdown="1">
For a commutative ring, we define:

1. An element $a$ is called a [_unit_](https://knowyourmeme.com/photos/1361701-absolute-unit) if there exists $b\in A$ such that $ab = 1$.
2. A _zero divisor_ is an element $a\in A$ such that there exists $b \in A$ and $ab = 0$, while $a \neq 0, b \neq 0$. Essentially, it's the paradoxical situation of multiplying two things to get nothing.
3. An element in a ring $a \in A$ is called _nilpotent_ if there exists some natural number $n>0$ such that $a^n = 0$.

</div>

A quick example in $\mathbb{Z}_4$ : 
- $3$ is a unit since $3 * 3 = 9 \ (mod \ 4) = 1$.
- $2$ is a zero-divisor (and a nilpotent) since $2 * 2 = 4 \ (mod \ 4) = 0$.

<div class="remark">
Note, also, that all nilpotents are zero divisors, but not all zero-divisors are nilpotents.
</div>

We will also use the following facts for a (commutative) ring $A$:

- **Fact 1** The Nilradical $Nil(A)$ is the ideal formed by all nilpotent elements in $A$.
- **Fact 2** $Nil(A)$ is the intersection of all prime ideals of $A$.
- **Fact 3** (Exercise 1) The sum of a unit and a nilpotent is a unit in $A$. (See [Solution](https://math.stackexchange.com/a/1957279/627540))
- **Fact 4** In an integral domain $I$, we have for $f,g \in I[x]$, $deg(fg) = deg(f) + deg(g)$. (See [Lemma 1.7, Notes:IntegralDomain](https://www.math.columbia.edu/~khovanov/ma2_fall/files/03_integral_domains.pdf)).

# Exercise 2. What happens in A[x]

Now that we know what units, zero divisors and nilpotents look like in the normal ring $A$, we look at how their equivalents  look like in the polynomial ring $A[x]$. This is the ring of _finite-degree_ polynomials $f(x) = a_0 + a_1x + \ldots + a_nx^n$, where the coefficients $a_i \in A$, $\forall i \leq n$.

## Part 1. Units of A[x]

Suppose that $f(x) \in A[x]$ as $f(x) = a_0 + a_1x + a_2x^2 + \ldots + a_n x^n$. We need to show that $f(x)$ is a unit $\iff$ $a_0$ is a unit in $A$ and $a_1,\ldots,a_n$ are nilpotent.

$\implies$ Assume that $f(x)$ is a unit. By definition, we have that 

$$
\begin{aligned}
f(x)g(x) & = (a_0 + a_1 x + \ldots + a_n x^n)(b_0 + b_1 x + \ldots + b_mx^m) = 1 \\
\implies & a_0b_0 = 1, \\
         & (a_1b_0 + a_0b_1) = 0 \\
         & \ldots \\
         & a_n b_m = 0
\end{aligned}
$$

and so on. This just follows from writing out the coefficients of $x^k$, $0 < k \leq m+n$. More concisely, we have that $\forall 0 < k \leq m+n$, $\sum_{i+j = k} a_ib_j = 0$. Let's use this. For example, we have that the highest power term $a_n b_m = 0$. Then, 
$$
\begin{aligned}
    & a_{n-1}b_{m} + a_{n}b_{m-1} = 0 \\
    & \implies a_{n}a_{m-1}b_{m} + a^2_{n}b_{m-1} = 0 \\
    & \implies a^2_{n}b_{m-1} = 0 \\
\end{aligned}
$$

since $a_{n}b_{m}$ zeroes out in the first term. Let's set up an induction on $r$: the hypothesis is that $\forall r$, $a_n^{r}b_{m-1} = 0$. We've just shown the base case. Then, we have the coefficient of $r-1$ as : 

$$
\begin{aligned}
    & a_{n-r-1}b_{m} + a_{n-r-2}b_{m+1} + \ldots + a_{n}b_{m-r-1} = 0 \\
    & \implies a^r_{n} \left( a_{n-r-1}b_{m} + a_{n-r-2}b_{m+1} + \ldots + a_{n}b_{m-r-1} \right) = 0 \\
    & \implies a^{r+1}_{n}b_{m-r-1} = 0
\end{aligned}
$$

So, carrying the induction all the way to $r = m-1$, we obtain that $a_n^{m-1}b_{0} = 0$, which is not possible. The contradiction comes from the fact that $a_{0}b_{0} = 1$, and so $b_{0}$ is a unit. Therefore, it cannot be a zero divisor, and in particular, $a_{n}$ is nilpotent. 

We can leverage this by noticing that $f(x) - a_{n}x^{n}$ is the difference of a unit (by hypothesis) and a nilpotent element (which we just proved). By Fact 3 we deduce that 

$$
\begin{aligned}
    f(x) - a_{n}x^{n} = a_{n-1}x^{m-1} + \ldots + a_0 
\end{aligned}
$$

is also a unit. We can thus descend on the degree of $f$, showing at each level by the same argument that $a_{n-1}, a_{n-2}, \ldots, a_1$, must all be nilpotent. This finishes the forward path. 

$\impliedby$ Suppose then that $f(x) = a_{0} + a_{1}x + \ldots + a_{n}x^{n}$ is a polynomial such that $a_{0}$ is a unit, and $a_{1}, \ldots$ are all nilpotent. Then, we have that $a_1x, \ldots, a_nx^n$ are the sum of nilpotent elements, and are thus nilpotent themselves. The fancy way of saying this is that the Nilradical (ideal of all nilpotent elements) is (shocker) an ideal. Therefore, $f(x)$ is the sum of a unit ($a_0$) and a nilpotent, and is thus a unit (following from Exercise 1).

## Part 2: Nilpotents in A[x]

Now, we examine the nilpotent elements of $A[x]$. In particular, we need to show that $f(x) = a_0 + \ldots + a_nx^n$  is nilpotent if and only if all the coefficients are nilpotent. 

$\implies$ Suppose then that $f$ is nilpotent. Then there must be some $k\in\mathbb{N}$ such that $f(x)^k = 0$. This implies that $a_n^k x^{nk} = 0$ i.e. $a_n$ is nilpotent. So, looking as usual on the difference 

$$
\begin{aligned}
f(x) - a_nx^n = a_{n-1}x^{n-1} + \ldots + a_{0}
\end{aligned}
$$

we have that the LHS is nilpotent (sum of nilpotents is nilpotent). Therefore, we can inductively show that for each $a_i$, it is nilpotent by descending on the degree of $f$.

$\impliedby$ This just follows from the fact that the polynomial expressed as the sum of nilpotents.

## Part 3: Zero-Divisors in A[x]

Now, we show that $f(x)$ is a zero-divisor in $A[x]$ $\iff$ there exists some $a\in A$ such that $af(x) = 0$ i.e. some constant annihilates all the ring. Notice that the reverse implication trivially follows from the definitions, so we forget about it.

$\implies$ Suppose there exists some $g(x)\in A[x]$ of least degree $m$ such that $f(x)g(x) = 0$. Then, we must have that $a^nb^m = 0$, by writing out the multiplication. In particular, if I replace $g(x)$ with $a_n g(x)$, we have that $f(x)\cdot a_ng(x) = a_nf(x)g(x) = 0$, since multiplication is commutative. However, notice that $a_ng(x)$ has degree $m-1$, since $a_n$ annihilates $b_m$. Since $g$ was supposed to be the minimial degree annihilator, we must have that $a_ng(x) = 0$. 

Okay, we're getting somewhere. We can repeat this process for $a_{n-r}$, $r\leq n$ to show that for all $a_i$, $a_ig(x) = 0$. Observe now that we have $a_ib_m = 0$, for all $i \leq n$ and this gives us the assertion that $b_m f(x) = 0$. This concludes the proof.

## Part 4: Primitives in A[x]

So, the exercise introduces primitive polynomials at this point. These are defined as polynomials $f(x) \in A[x]$ such that the ideal generated by the coefficients generates the ring i.e. $(a_0, a_1, \ldots, a_n) = (1) = A$. For a quick example, consider $A = \mathbb{Z}$ and $f(x) = 1 + 3x$.

We need to show that $f(x)$ and $g(x)$ are primitive $\iff$ $fg$ is primitive. This is where I start getting tired of chasing coefficients, and start embracing the algebra.

$\implies$ Suppose by contradiction that $fg$ is not primitive. Then, its coefficients $(c_0, c_1, \ldots, c_{n+m})$ must be contained in some maximal ideal $M$ (which is by definition proper). Since $M$ is maximal in $A$, we know that $A/M$ is a field (and in particular, an integral domain). Then, looking in the ring $(A/M)[x]$ (the quotient map sends polynomials $A[x] \rightarrow (A/M)[x]$), we have that $fg(x) = 0$; however, $f(x) \neq 0, g(x) \neq 0$. Since an integral domain does not have any zero-divisors, this is a contradiction.

$\impliedby$ To show the other way, suppose that $fg$ is primitive, and $f$ is not primitive. Then, the coefficients of $f$, $(a_0, \ldots, a_n) \subseteq M$, where $M$ is some maximal ideal, again. In particular, the coefficients of $fg$ are contained in the product ideal $(a_0, \ldots, a_n)(b_0, \ldots, b_m) \subseteq (a_0, \ldots, a_n) \cap (b_0, \ldots, b_m) \subseteq M \cap A = M$. However, that contradicts the primitivity of $fg$. Reasoning similarly for $g$, we conclude the proof.

## Retour en arrière ...

Notice that Part 4 involved absolutely no coefficient chasing. Instead, we just used ring-theoretic properties to a few contradictions. Being very concise, it trades off the "first-principles" interpretability for ease of proof. Something I have no doubt will follow me for the rest of the chapter. In fact, we can re-use this type of thinking to shorten the longest proof, Part 1's forward implication.

**(Part 1, v2)** 
$\implies$ Suppose that $f(x)$ is a unit i.e. $f(x)g(x) = 1$ for some $g(x)$. Then, we take some prime ideal $p$ and look modulo the quotient $A/p$. This gives us $\tilde{f}(x)\tilde{g}(x) = 1$ (in A/p[x]). Since $A/p$ is an integral domain, we must have that $deg(\tilde{f}\tilde{g}) = 0 = deg(\tilde{f}) + deg(\tilde{g})$, and in particular, $deg(\tilde{f}) = 0$ (Fact 4). This forces $\forall 1 \leq i \leq n$, $\tilde{a_i} = 0$. Since the choice of prime ideal was arbitrary, we must have that $a_i$ ($i \geq 1$), must be $0$ in all prime ideals, and in particular, they belong to them all. Alternatively, they belong to $Nil(A)$ (Fact 2), concluding the proof.


# Conclusions

- Working through [AM](https://www.taylorfrancis.com/books/mono/10.1201/9780429493638/introduction-commutative-algebra-michael-atiyah-macdonald) is kind of nice. Definitely need to pair this with other material to make sure I'm doing the right things.
- Learning (what I call) the algebraic way of doing things, as opposed to starting from the definitions, is an exercise in and of itself. It can be incredibly opaque. Going to re-visit this when I finish the chapter exercises.

# Important Resources:

- [Pete L. Clark's Notes](https://plclark.github.io/PeteLClark/Expositions/integral2015.pdf) on Commutative Algebra. More of a reference text, but definitely less terse than AM.
-  Good Ol' [StackExchange](https://math.stackexchange.com).
