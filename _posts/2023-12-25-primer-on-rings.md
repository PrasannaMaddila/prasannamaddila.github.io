---
layout: post
title: "An Introduction to Rings"
categories: math
---

This article will motivate the example of a ring, a common algebraic structure, using various examples. Briefly, we basically want to see what happens when we add a second operation to a group.

## Integers

This is the canconical ring in some sense since the integers $\mathbb{Z}$ are typically associated with two operations $+$ and $\times$. In addition, $(\mathbb{Z}, +)$ is a commutative group, while $(\mathbb{Z},\times)$ doesn't even have inverses! However, these two operations are also compatible with each other via the distribitivity axioms, $\forall a, b, c \in \mathbb{Z}, a(b+c) = ab + ac$ and $(a+b)c = ac + bc$. Somewhat interestingly, this is a common structure. 

## Square Matrices

In this case, we consider the space of $n \times n$ matrices over $\mathbf{R}$, for instance. We denote this space as $M_{n\times n(\mathbf{R}}). Adding matrices is simply defined as 
$$
\begin{align}
    A + B &= \left[ a_{ij} \right]_{n \times n} + \left[ b_{ij} \right]_{n \times n} =\left[ a_{ij} + b_{ij} \right]_{n \times n} 
\end{align}
$$

i.e. it is element-wise addition. Clearly, we immediately get associativity as
$$
\begin{align}
    (A + B)+C &= ( \left[ a_{ij} \right]_{n \times n} + \left[ b_{ij} \right]_{n \times n} ) +  \left[ c_{ij} \right]_{n \times n} \nonumber \\
    &= \left[ a_{ij} + b_{ij} \right] + \left[ c_{ij} \right] \nonumber \\
    &= \left[ a_{ij} + b_{ij} + c_{ij} \right] \nonumber \\
    &= \left[ a_{ij} + (b_{ij} + c_{ij}) \right] \nonumber \\
    &= \left[ a_{ij} \right] + \left[ b_{ij} + c_{ij} \right] \nonumber \\
    &= A + (B+C)
\end{align}
$$

where we used the associativity of $\mathbb{R}$ and the definition of the operation. Similarly, we can show that the zero matrix $\mathbf{0}$ is the additive identity, and for each matrix $A$, $-A$ is the additive inverse. It is also trivially commutative under addition as well. 

For the multiplication operation, we consider regular matrix multiplication. It admits an multiplicative identity $\mathbf{I}$, but we cannot guarantee that each matrix has an inverse. We can quickly show distributivity in Einstein notation as
$$
\begin{align}
D = A(B+C) &\implies d_{ik} = a_{ij} (b_{jk} + c_{jk}) = a_{ij}b_{jk} + a_{ij}c_{jk} = AB + AC
\end{align}
$$

as desired. Note however that this does not really tell us anything interesting, and proofs viewing matrices as linear transformations might be worth a look.

In short, the space of square matrices is a ring under element-wise addition and matrix multiplication.


## Continuous Functions

Finally, we consider the space of all continuous real-valued functions, $\mathcal{C}(\mathbb{R}) = \{ f: \mathbb{R} \rightarrow \mathbb{R} \}$. Under pointwise addition, i.e. $(f+g)(x) = f(x) + g(x)$, we can easily see that it forms a commutative group. The constant function $f_0(x) = 0, \forall x$, is the additive identity, and the constant function $f_1(x) = 1, \forall x$, is the multiplicative identity. Distributivity is also quite natural to show, since we always have for example, $f(x) (g(x) + h(x)) = f(x)g(x) + f(x)h(x)$, for any three such functions $f,g,h$. However, the inverse of a function is not necessarily defined, since any function with a zero i.e. $\exists x \in \mathbb{R}$ such that $f(x) = 0$, cannot have a (pointwise) inverse. 


## General Definition

To talk about all these examples at once, we consider a definition.

<div class='defn' text='Rings'>
    A ring $(R, +, \times)$ is a set endowed with two operations such that
    <ol>
        <li> $(R, +)$ is a commutative group. </li>
        <li> $(R,\times)$ is associative i.e. $a(bc)=(ab)c=abc$, $\forall a,b,c \in R$. </li>
        <li> $+$ and $\times$ distribute over each other i.e. $a(b+c) = ab + ac$ and $(a+b)c = ac + bc$, for all $a,b,c \in R$.</li>
    </ol>
</div>

Note that we implicitly assume that the ring is commutative ( $ab = ba, \forall a,b \in R$ ) and has a multiplicative identity. But with that, we know exactly what a (commutative) ring is !
