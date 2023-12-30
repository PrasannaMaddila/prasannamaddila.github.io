This summarises a few definitions and theorems involving groups. The goal is to briefly introduce enough content, so that we can quickly move on to rings. Consider the basic definition of a group.


### Groups
<div class="defn">
A group is a set $G$ equipped with a binary mapping, $\circ$ such that it satisfies the following properties: 
<ol>
<li> Closure : $\forall x, y \in G, xy \in G$ </li>
<li> Existence of Identity : $\exists e \in G$ such that $\forall x \in G, xe = ex = x$, and </li>
<li> Existence of Inverses : $\forall x \in G, \exists y \in G$ such that $xy = e$.</li>
</ol>
</div>

We can give simple examples for groups, such as the integers under addition $(\mathbb{Z}, +)$, or the modular integers $(\mathbb{Z}\_n, +)$ where the operation is $x \rightarrow x / mod / n$. A small note, we denote a group $(G, +)$ as simply $G$, whenever it is obvious.

Note that we leave out many common assumptions, such as commutativity. While the group $(\mathbb{Z}, +)$ is commutative, interesting non-commutative examples include the group of $n \times n$ matrices $M_{n\times n}(\mathbb{R})$, or the symmetry group on $n$ objects $S_n$. Interestingly, a subgroup is a subset $H \in G$ that is closed under $\circ$. We can also define more interesting operations on a group such as quotients, and define mappings between them. In particular, we have homomorphisms and isomorphisms.

### Cosets and Quotients

<div class="defn">
A (left) coset of a group $G$ is some subset $C \subset G$ such that $\forall a \in G, b \in C$ we have that $ab \in C$. Similarly, a right coset is defined a subset for which the rule is $ba \in C$.
</div>

Note that if a group is commutative, the definitions coincide, and we can speak about cosets without distinguishing the left-iness or the right-iness. An interesting operation using this coset structure would be $(g_1H)(g_2H) = g_1g_2 H$, where $g_1H$ and $g_2H$ are two cosets induced by $H$. Let us expand this definition to see what it implies.
$$
\begin{align*}
    \forall h_1, h_2 \in H, (g_1g_2H) &= (g_1H)(g_2H) = (g_1h_1 H)(g_2h_2 H) = (g_1h_1g_2h_2)H \\
    \implies& (g_1g_2)^{-1}(g_1h_1g_2h_2) \in H \\
    \implies& (g_2^{-1}h_1g_2h_2) \in H
\end{align*}
$$

In particular, we require that $g_2^{-1}h_1g_2 \in H$ for all $g_2 \in G, h_1 \in H$ for this operation to be well-defined. We call this condition Normality, and we denote a (sub)group that obeys this rule a __normal__ (sub)group. 

This is precisely the structure we need to define a group quotient, which is a way of dividing a group. Intuitively, given a normal subgroup $H \subset G$, we can define a new group $G/H$ where every element is then taken 'modulo' $H$. Normalilty means that the natural group operation is well-defined, so we can justify the following definition.

<div class="defn">
Given a group $G$ and a normal subgroup $H \in G$. Then, we define the group quotient $G/H$ as a new group where 
<ol>
<li> $G/H = \{ H, a_1 + H, a_2 + H, \cdots \}$ are the elements </li>
<li> The group operation $\circ'$ is defined as $(g_1H)(g_2H) := (g_1g_2)H$. </li>
</ol>
</div>


### Homomorphisms
The other natural operation on groups is to define mappings between them. In particular, we are interested in mappings that preserve the group-iness i.e. mappings between sets are not all too interesting. This gives us a very natural definition.

<div class="defn">
A group homomorphism $f: (G,+) \rightarrow (H, \circ)$ is a mapping between two groups such that $f(a + b) = f(a) \circ f(b)$. 
</div>

In particular, preserves the group structure since it (intuitively) translates the group nature of $G$ into that of $H$. Whenever $f$ is also a bijection, it is called an __isomorphism__. When two groups $G,H$ are isomorphic, we denote it as $G \cong H$. Intuitively, we can consider them both as equivalent. however disparate their actual structure. 

Funnily enough, both quotient-ing and homomorphisms are quite related, at least by the first Isomorphism Theorem.

<div class="thm">
Let $f: G \rightarrow H$ be a group homomorphism. Then, we have the following.
<ol>
<li> Let the kernel of $f$ be the set of elements of $G$ mapped to the identity element of $H$, defined as $ker(f) = \{ g \in G : f(g) = e_H \}$. Then, $ker(f)$ is a normal subgroup of $G$. </li>
<li> $G / ker(f) \cong Image(f)$. In particular, if $f$ is surjective, $Im(f) = H$ and we have $G\ker(f) \cong H$. </li>
<ol>
</div>

In summary, a homomorphism induces a normal subgroup (the kernel of this homomorphism), and modulo this subgroup, we get back (more or less) $H$. To be specific, we get back something that is isomorphic to $H$.
