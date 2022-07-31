---
layout: post
title:  "Minkowski sums and the GJK algorithm"
post_image: '<img src="\assets\img\2021-08-01-gjk-algorithm\preview.png"  style="width:100%; display: block; margin-left: auto; margin-right: auto;" >'
---

The Gilbert-Johnson-Keerthi algorithm is a fast and efficient way to compute a pair of closest point between two convex objects, and - in particular - to find out if they have a common intersection or how far from each other they are. The main idea on which the algorithm is based is simple and beautiful, as it uses the concept of Minkowski sum (difference, in this case) to transform and simplify a problem between two bodies in a problem between a single body and a point.

# What is a Minkowski sum?

Given two convex bodies $$P$$ and $$Q$$ in $$\mathbb{R}^d$$, their *Minkowski sum* is the set of all the points of $$\mathbb{R}^d$$ which can be expressed as the sum of a point in $$P$$ with a point in $$Q$$. This sum is denoted with the symbol $$\oplus$$, so in math notation we could write it as

$$ P \oplus Q :=  \left\{ p + q \; | \; p \in P, q \in Q \right\}. $$

Putting aside its location in space, one can intuitively think of the Minkowski sum of $$P$$ and $$Q$$ as the region swept out by moving $$Q$$ along $$P$$. We are drawing a $$P$$ shape using a $$Q$$-shaped brush instead of a pencil! Of course one can switch the role of $$P$$ and $$Q$$ and the result would be the same. A two dimensional example is represented below. 

<img src="\assets\img\2021-08-01-gjk-algorithm\minkowski_sum.svg"  style="width:75%; display: block; margin-left: auto; margin-right: auto;" >

Minkowski sums of convex polytopes are meaningful objects in mathematics. They can for example be used to count solutions of polynomial systems (a result by O. Gelfond and A. Khovanskii), by associating each polynomial to a polytope, and then studying the properties of their Minkowski sum. I dealt with them myself during my years in academia (see [here](https://arxiv.org/pdf/1904.01343.pdf), where you can find some nice pictures of some Minkowski sums of three dimensional polytopes).

# The main idea

The GJK algorithm is based on the following idea. Suppose that two sets $$P$$ and $$Q$$ in $$\mathbb{R}^d$$ have a common intersection, this is equivalent to say that there is a point $$p$$ in $$P$$ and a point $$q$$ in $$Q$$ such that $$p = q$$, or, equivalently, $$p - q = \mathbf{0}$$. The bold zero is to emphasize that we are talking of points, not numbers. In particular $$\mathbf{0}$$ is the *origin* of the space where $$P$$ and $$Q$$ live. The previous equation means that the origin $$\mathbf{0}$$ can be written as a sum of two points: $$p$$ in $$P$$ and $$-q$$ in $$-Q$$ (this is the *reflection* of $$Q$$, obtained by inverting the sign of all its points). But if this is true then this means that the origin $$\mathbf{0}$$ is in the Minkowski sum of $$P$$ and $$-Q$$. Sometimes it is easier to see this in math terms:

$$
\begin{array}{llll}
P \cap Q \neq \varnothing   & \iff & p = q                          & \text{for some points $p \in P$, $q \in Q$} \\
                            & \iff & p - q = \mathbf{0}             & \text{for some points $p \in P$, $q \in Q$} \\
                            & \iff & p + (-q) = \mathbf{0}          & \text{for some points $p \in P$, $q \in Q$} \\
                            & \iff & \mathbf{0} \in P \oplus (-Q).  & \\
\end{array}
$$

The Minkowski sum of $$P$$ and $$(-Q)$$ is often called the *Minkowski difference* of $$P$$ and $$Q$$, and it is denoted by $$\ominus$$.
Note that these are all equivalences, which is to say that $$P$$ and $$Q$$ have a common intersection *if and only if* the origin is in their Minkowski difference.

Note how translating $$P$$ and $$Q$$ influences how the origin is in their Minkowski difference or not.

<img src="\assets\img\2021-08-01-gjk-algorithm\polygons_without_intersection.svg"  style="width:75%; display: block; margin-left: auto; margin-right: auto;" >

<img src="\assets\img\2021-08-01-gjk-algorithm\polygons_with_intersection.svg"  style="width:75%; display: block; margin-left: auto; margin-right: auto;" >

Note that in what we described above we have not even required convexity for $$P$$ and $$Q$$, we have not even used the fact that they are polytopes, they might just have been any pairs of subsets of the real space. Nevertheless dealing with Minkowski sums (or even checking if $$\mathbf{0}$$ is contained in them) is really tough business if we don't assume the $$P$$ and $$Q$$ are convex polytopes (or polyhedra, if you prefer - which you should not[^polyhedra]).

If $$P$$ and $$Q$$ are polytopes then we can note that the vertices of their Minkowski difference are differences of vertices of $$P$$ and $$Q$$[^verticesMink]. Suppose that $$P$$ has $$n$$ vertices and $$Q$$ has $$m$$ vertices. This means that we could try to look for the vertices of $$P \ominus Q$$ simply by looking among the $$n * m$$ possible differences of the vertices of $$P$$ and $$-Q$$. This leads to two problems. First, we do not know how verify that a difference of two vertices will be a vertex of $$P \ominus Q$$. Secondarily we have a complexity problem, as $$n * m$$ points to deal with might be too many. We "solve" the first problem by restricting ourselves to a convex setting, i.e. we assume that both $$P$$ and $$Q$$ are convex polytopes, so that $$P \ominus Q$$ will also be a convex polytope[^convMink]. An approach would be to find which of the $$n * m$$ possible points are actually the vertices of $$P \ominus Q$$. A convex hull algorithm would do exactly this, but only doing that has complexity $$O(k \log k)$$, where $$k$$ is the number of points. The idea is to check if $$\mathbf{0}$$ is in $$P \ominus Q$$ using as few of the vertices of the Minkowski sum as possible, and track down its "origins" (by this I mean the vertices of $$P$$ and $$Q$$ it comes from) as rarely and efficiently as possible.

The algorithm requires the use of *support* function. Given a direction vector $$d$$ we call $$\mathrm{supp}_P(d)$$ to be the point of the polytope $$P$$ which maximizes the dot product with $$d$$. Informally, this means that $$\mathrm{supp}_P(d)$$ is "as far away as possible" along the direction of $$d$$.  We are gonna use the fact that the support function behave well with Minkowski sums of convex sets, meaning that we can distribute the support over $$P \ominus Q$$ over $$P$$ and $$Q$$ as follows

$$ \mathrm{supp}_{P \ominus Q}(d) = \mathrm{supp}_{P}(d) + \mathrm{supp}_{-Q}(d) = \mathrm{supp}_{P}(d) + \mathrm{supp}_{Q}(-d). $$

# The algorithm

**Variables**
- $$d$$ stores a direction vector,
- $$p$$ stores the support of $$d$$ over $$P \ominus Q$$
- $$S$$ stores a simplex[^simplex], its dimension will change during the algorithm.

**Algorithm**
1. Initialize the variables:
    - fix any direction $$d$$,
    - let $$p = \mathrm{supp}_{P \ominus Q}(d)$$, fix $$S = \{ p \}$$.
2. Find the point $$x$$ of minimum norm (= distance from the origin) of $$S$$.
    - If $$x = \mathbf{0}$$ then return $$P$$ and $$Q$$ as intersecting.
3. Update the variables:
    - $$d = -x$$,
    - $$p = \mathrm{supp}_{P \ominus Q}(d)$$.
4. Check that $$p$$ is more extremal in direction $$d$$, i.e. if $$ p \cdot d > x \cdot d $$.
    - If not, then return $$P$$ and $$Q$$ as non intersecting and with distance equal to the norm of $$x$$.
5. Reduce $$S$$ to the smallest subset $$S' \subset S$$ such that $$x$$ is in the convex hull of $$S'$$. Add $$p$$ to $$S$$ and go to 2.



<img src="\assets\img\2021-08-01-gjk-algorithm\gjk_algorithm.svg"  style="width:75%; display: block; margin-left: auto; margin-right: auto;" >

# Remarks

 - The algorithm can be extended to general convex sets, provided that we can easily evaluate support functions over them (for example it can be used to calculate sphere-polytope intersections).
 - Note that supports on polytopes can be calculated by brute force in linear time (with respect to the number of vertices of the polytope), but if information for the edges is available, this can be sped up with a [simplex algorithm](https://en.wikipedia.org/wiki/Simplex_algorithm) (sometimes also called *hill-climbing* algorithm).



---

[^polyhedra]: Usually (at least in mathematics) *polytopes* and *polyhedra* are both names for convex bodies obtained as the intersections of half-spaces, but polytopes have the additional assumption of being bounded regions.

[^verticesMink]: This seems obvious once one has been playing a bit with Minkowski sums, but a rigorous proof is consequence of the fact that each linear function over a polytope is always maximized by a vertex, and when evaluating a linear function over a Minkowski sum, one can always distribute it over its Minkowski summands.

[^convMink]: Again, this seems obvious, but it needs a rigorous proof. Assuming that we are using the "usual" definition of convexity (a segment between two point is entirely contained), one proves it by describing a segment into a linear parametric function and distribute it over the Minkowski summands.

[^simplex]: A simplex is the convex hull of $$d+1$$ affinely independent points in $$d$$-dimensional space, for example, a $$0$$-simplex is a point, a $$1$$-simplex is a line segment, a $$2$$-simplex is a triangle, and a $$3$$-simplex is a tetrahedron.
