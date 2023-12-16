# Primer on Rings 

This summarises the main definitions and theorems involving rings - at least the ones involved in an introductory class.

Consider the basic definition of a group.

### Groups

> A group is a set $X$ equipped with a binary mapping, $+$ such that the following properties are satisfied. 
    - Closure : $\forall x, y \in X, x+y \in X$.
    - Existence of Identity: $\exists e \in X$ such that $\forall x \in X, x+e = e+x = x$
    - Existence of Inverses: $\forall x \in X, \exists y \in X$ such that $x + y = e$

We can give simple examples for groups, such as the integers under addition $(\mathbb{Z}, +)$, or the modular integers $(\mathbb{Z}\_n, +)$ where the operation is $x \rightarrow x mod n$.
