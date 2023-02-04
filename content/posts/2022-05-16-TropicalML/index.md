---
title: "Piecewise linear activations: Machine Learning meets Tropical Geometry (Part 1)"
date: 2022-05-16T21:21:45+01:00
tags: ["machine learning", "geometry", "tropical geometry"]
math: true
---

Activation functions decide what is to be fired to the following neuron in a neural network depending on the input signal received. Since they generally are the only nonlinear component in a model architecture, they are what truly allows networks to learn complex patterns, as without them network would only approximate linear functions (and most of their structure would be redundant). Although traditionally the sigmoid was the most common choice for an activation function, in modern neural network the default recommendation is to use the *Rectified Linear Unit* (ReLU) defined as $x \mapsto \max(0,x)$, or a more general *Leaky ReLU* $x \mapsto \max(ax,x)$, or a even more general *Maxout unit*. All these activation functions are *piecewise linear*, meaning that their graphs are composed of straight-line segments.

![activation_functions](activation_functions.svg#center "Some of the most common activation functions.")

Using a piecewise linear activation function makes the network a piecewise linear function, i.e. the input space gets partitioned in polyhedral regions, and in each region the network behaves linearly. 

Studying such linear regions is an important step in the understanding of deep neural networks: since their goal is to be good approximators, their number of linear regions is a good measure of their *expressiveness*. In recent years a lot of progress to find lower and upper bounds for this quantities has been made (see [this paper by Montúfar et al.](https://arxiv.org/abs/2104.08135) for some general and sharp upper bounds). Note that the number of regions gets quickly very large: lower bounds tells us that there are at least $10^{80}$ even for small architectures. If you compare this number with the general number of training samples ($\leq 10^6$), it is clear that **basically none of the linear regions contains training data**!

> This raises the question of how representative the number of linear regions is for network performance and how information extracted from training samples passes on to the many linear regions free of data for successful generalization to test data (from [Trimmel et al.](https://openreview.net/forum?id=IqtonxWI0V3)).

An important tool for this kind of investigations has been Tropical Geometry. It turns out that **the family of ReLU/Maxout neural network functions corresponds with the family of tropical rational maps** ([Zhang et al.](https://arxiv.org/abs/1805.07091)). Now, once one familiarizes both with the structure of these networks and the theory of Tropical Geometry, it turns out that this result is quite straightforward. The key of this connection is instead that tools which are used to deal with tropical varieties can now be used to investigate the decision boundaries of neural networks, and observe how training affects them.

## An (illustrated) introduction to Tropical Geometry
Tropical Geometry is a relatively recent field of study in mathematics - laying somewhere between algebraic geometry and combinatorics - where the traditional study of algebraic varieties is substituted by the study of some polyhedral objects, defined by interpreting the polynomials over the *tropical semiring*. The best reference for tropical geometry is the book *Introduction to Tropical Geometry* by Diane Maclagan and Bernd Sturmfels. In it you will find the slogan "a tropical variety is a *combinatorial shadow* of the original variety", which is now is repeated as a mantra in paper and conferences. There are 1870 results on Google for ["combinatorial shadow of"](https://www.google.com/search?q=%22combinatorial+shadow+of%22), probably all from this context (edit from late 2022: they doubled!). Remember to use it if you write a paper on this subject.

Another curiosity is the origin of the term *tropical* in Tropical Geometry. From the aforementioned book:

> The adjective “tropical” was coined by French mathematicians, notably Jean-Eric Pin, to honor their Brazilian colleague Imre Simon, who pioneered the use of min-plus algebra in optimization theory. There is no deeper meaning in the adjective “tropical”. It simply stands for the French view of Brazil. 

### Basic definitions

The *tropical semiring* is defined as $\mathbb{T} := (\mathbb{R} \cup \\{ -\infty \\}, \oplus, \odot) $ where

 - $x \oplus y := max\\{ x, y \\}$ is the *tropical addition*,
 - $x \odot y := x + y$ is the *tropical multiplication*.

The extra element $-\infty$ has been added to ensure that $\mathbb{T}$ has an additive identity, i.e. $x \oplus -\infty = x$, while $0$ is the multiplicative identity. This definition ensures associativity, commutativity and distributivity, although the addition is not invertible, for example $x \oplus 1 = 0$ has no solutions. For this reason $\mathbb{T}$ is a semiring, not a ring. Note that $\min$ tends to be used more frequently instead of $\max$ by mathematicians, but since it is only a matter of conventions we are going to use the $\max$-convention, as it will adapt better to the machine learning context. In any case, the whole theory is equivalent regardless of this choice.

Since we have a multiplication operation, it makes sense to talk about exponentiation, defining it as an iterated multiplications. Although we might use the notation $x^{\odot 2}$ to indicate $x \odot x$, we will often just write it $x^2$, especially if it is clear that we are on a tropical context.

### Roots of tropical polynomials

If interpreted as functions over the tropical semiring, polynomials are piecewise linear functions, where the "pieces" - or *linear regions* - are polyhedral. Consider the univariate tropical polynomial

$$
    p(x) = x^2 \oplus 2 \odot x \oplus 1 = \max \\{ 2x, x + 2, 1 \\},
$$

in this case the linear regions are the intervals $(-\infty,-1], [-1,2], [2,+\infty)$. Here the values $-1$ and $2$ play a special role, as each of them belongs to two intervals. Moreover, they appear explicitly if we rewrite $p$ as $(x \oplus -1) \odot (x \oplus 2)$. It makes then sense to call them *tropical roots* of $p$, in analogy with what happens in the traditional settings.

Note that, as in the example below, the factorizations (and therefore the roots) in the tropical and classical settings are different.

![polynomial_factorization](polynomial_factorization.svg#center "The same polynomial has different factorizations if interpreted in classical or in tropical setting.")

The fact that a tropical polynomial can be factored as in the example above is a general property of tropical univariate polynomial functions, which means that the so-called *Fundamental Theorem of Algebra* holds tropically. Note the word "polynomial *function*". Indeed different polynomials define the same functions, just because some summands may play no role. For example, in the polynomial function defined by the polynomial

$$
    q_a(x) = x^2 \oplus a \odot x \oplus 0 = \max \\{ 2x, x + a, 0 \\}
$$

the term $a \odot x$ plays no role whenever $a \leq 0$. In particular, even being different as polynomials (because they have different coefficients), $q_{-1}(x)$ and $q_0(x)$ define the same polynomial function $x \mapsto \max(2x, 0)$. So the correct way to state the Tropical Fundamental Theorem of Algebra is that *every tropical univariate polynomial can be replaced by an equivalent polynomial (defining the same polynomial function) that can be factored - in a unique way - into linear factors*.

To summarize, for univariate tropical polynomials, the roots are those elements in the domain belonging at least to two linear regions (or, equivalently, for which the maximum is attained at least twice) and, to confirm that this definition is meaningful, they appear in the "unique" factorization of the polynomial.

### Tropical curves, surfaces and hypersurfaces

But what happens in higher dimensions, when we have more variables other than just $x$? In the classical setting, the concept of roots is extended as the *vanishing locus* or the *zeroes* of a polynomial, i.e. all the points in the domain where the polynomial evaluate to zero. These sets are called *algebraic curves* in $\mathbb{R}^2$ (such as an ellipse) for bivariate polynomials, *algebraic surfaces* $\mathbb{R}^3$ (such as a sphere) for trivariate polynomials, or - more generally - *algebraic hypersurfaces* when the dimension is higher or non specified. In the tropical setting the concept of tropical roots is extended by focusing on the linear regions: **given a polynomial, the *algebraic tropical curve/surface/hypersurface* defined by the polynomial is the set of all the points belonging to at least two linear regions**. 

Here is an example of how (the function defined by) a degree two tropical polynomial in two variables looks like.

![tropical_polynomial](tropical_polynomial.svg#center)

And here is the algebraic tropical *quadric* (curve of degree two) it defines. In a way, the tropical curve is what you see when you look at tropical function "from above".

![tropical_curve](tropical_curve.svg#center60)

As we increase the degree of the polynomials, curves get more complex. Here is what a degree fifteen tropical curve with random integer coefficients looks like.

![tropical_curve_random](tropical_curve_random.svg#center60)

And here is an example of a tropical *surface* (image from [this paper](https://arxiv.org/abs/1806.02236)).

![quartic_surf](quartic_surf.svg#center60)

There are several clues that confirm that with these definitions we are onto something. Every mathematician's favorite example is the tropical version of ***Bézout's Theorem***. In the classical settings it states that two algebraic curves defined by polynomials of degrees $c$ and $d$ intersects in $cd$ points. Here the result is loosely stated, as it holds for "generic" curves defined on an algebraically closed field, and one need a notion of multiplicity when there is tangency. Usually it is stated for projective curves, as it takes a cleaner form. Anyway it should not sound too weird: two lines intersect in one point, a line and an ellipse intersect in two (possibly complex) points, two ellipses intersect in four (possibly complex) points, and so forth. The very same theorem holds tropically, and - if possible - it holds in a cleaner way, meaning that we do not need extra hypotheses such as working with complex numbers, or assuming that the curves have been chosen "generically" enough. All we need a concept of *stable* intersection to handle the cases where two curves are aligned (if you want to know more, have a look to Corollary 1.3.4 in the referenced book).

![bezout](bezout.svg#center "An illustration of Bézout's Theorem. A line intersecting a cubic in three points, both in classical and tropical setting.")

### Newton polytopes and Minkowski sums

We saw that for univariate polynomials there is unique factorization, but the same does not hold for two or more variables. The following are two *irreducible* factorizations of the same polynomial, i.e. which cannot be factorized any further.

$$
(x \oplus 0) \odot (y \oplus 0) \odot (x \odot y \oplus 0 ) = (x \odot y \oplus x \oplus 0) \odot (x \odot y \oplus y \oplus 0)
$$

One can simply expand the expression above and verify that it is true, anyway there is a much better way to get a geometric intuition of what is going on. In order to do so we introduce the concept of *Newton polytopes* and *Minkowski sums*.

**Given a polynomial in $n$ variables $f(x_1, \ldots , x_n)$, the *Newton polytope* $\text{Newt}(f)$ of $f$ is the convex hull of all the points $(a_1, \ldots, a_n) \in \mathbb{R}^n$ such that the monomial $x_1^{a_1} \cdots x_n^{a_n}$ appears in the expansion of $f(x_1, \ldots , x_n)$** with some coefficient.

As an example the polynomial

$$
0 \oplus 0 \odot x \oplus 0 \odot y \oplus 0 \odot x^2 y \oplus 0 \odot x  y^2 \oplus 0 \odot x^2 y^2 
$$

has the following hexagon as Newton polytope.

![newt_hex](newt_hex.svg#center60)

In a way the Newton polytope refines the notion of degree of a polynomial, by keeping track of the extremal exponents appearing in the polynomial. For example, for univariate polynomials the Newton polytope is simply a segment from the lowest exponent to the highest exponent.

When we multiply two polynomials, the resulting degree corresponds to the sum of the original degrees. In the case of the Newton polytopes we need to define a special sum that given two polytopes spits another polytope. **Given two polytopes $P$ and $Q$ we call their *Minkowski sum* $P + Q$ the set of all the points which can be expressed as the sum of a point in $P$ with a point in $Q$**, i.e.

$$ P + Q :=  \left\\{ p + q \\; | \\; p \in P, q \in Q \right\\}. $$

One can intuitively think of the Minkowski sum of $P$ and $Q$ as the region swept out by moving $Q$ along $P$ (or the other way around), as in the following picture.

![minkowski_sum](minkowski_sum.svg#center)

It is easy to see that this definition works well with Newton polytopes of product of polynomials. When we expand a product of two polynomials we compute - leaving coefficients aside - all the possible sums of one exponent of the first polynomial plus one exponent of the second. In other words, given two polynomials $f$ and $g$, **the Newton polytope of the product $f \odot g$ is the Minkowski sum of the Newton polytopes of $f$ and $g$**, which is 

$$
\text{Newt}(fg) = \text{Newt}(f) + \text{Newt}(g).
$$

We can now give a better explanation to the example above where a polynomial had multiple factorizations. If we take each of the Newton polytopes of each factor and we calculate their Minkowski sum, and we do this for both factorizations, we obtain the same Minkowski sum.

![double_minkowski](double_minkowski.svg#center)

### Subdivisions of Newton polytopes

Newton polytopes have much deeper connections with tropical polynomials than what we have seen so far. In the previous section we have ignored the role of coefficient of polynomial, as we have mostly focused on exponents only. Coefficient of a polynomial can be used to induce a *subdivision* of its Newton polytope, which is tightly related to the tropical hypersurface defined by the polynomial. The trick will be to add another dimension to the space where the Newton polytope lives, and store the coefficients in the extra coordinate.

Let $f(x_1, \ldots, x_n)$ be a polynomial in $n$ variables, we build the subdivision of its Newton polytope $\text{Newt}(P)$ in three steps.

1. Build the *lifted Newton polytope* $P_f$ as the convex hull of all the points $(a_1, \ldots, a_n, c) \in \mathbb{R}^{n+1}$ such that the monomial $x_1^{a_1} \cdots x_n^{a_n}$ appears in the expansion of $f(x_1, \ldots , x_n)$ with $c$ as a coefficient. From $P_f$ is an extended version of the Newton polytope $\text{Newt}(P)$, carrying extra information about the coefficients.
2. Isolate the *upper hull* of $P_f$, i.e. the set of its $n$-dimensional faces whose outer normals have positive last coordinate. This can be though as the part of the surface of the lifted Newton polytope which can be seen *from above*.
3. Project down the upper hull by dropping the last coordinate, inducing a subdivision of $\text{Newt}(P)$.

As an example, consider the polynomial 

$$
    f(x,y) = 0 \oplus 1 \odot x \oplus 1 \odot y \oplus 0 \odot x^2 \oplus 1 \odot xy \oplus 0 \odot y^2,
$$

then the lifted Newton polytope $P_f$ and the induced subdivision on $\text{Newt}(P)$ are depicted below.

![subdivision](subdivision.svg#center)
<img src="\assets\img\2022-05-16-TropicalML\subdivision.svg"  style="width:100%; display: block; margin-left: auto; margin-right: auto;" >

How is this subdivision related to the tropical hypersurface? Recall that a tropical polynomial defines a function which at each point selects the maximum of a set of affine linear maps evaluated at that point, for example the map defined by $1 \odot x^5 \oplus 4 \odot x^2 y^3$ selects the max between $1 + 5x$ and $4 + 2x + 3y$ at each point $(x, y)$. Well, each point of $P_f$ corresponds to a monomial of $f$, and each monomial of $f$ corresponds to an affine linear map. Moreover upper hull only selects the points/monomials/maps which play a role in the polynomial function, meaning that if a point is below the upper hull, then the corresponding affine linear map will never achieve the maximum (and so could be discarded). Moreover, faces of the subdivision correspond to affine linear maps attaining maxima for some common values.

Passing to the tropical hypersurface, **this geometric construction defines a beautiful duality between the subdivision and the hypersurface, which allows us to completely deduce the combinatorial structure of the hypersurface from the subdivision of its Newton polytope** (and the other way around). To each vertex of the subdivision correspond to a linear region, and each face of the subdivision correspond to a face of the hypersurface. Visualizing this in dimension two makes the duality much more evident:

 - each **polygon** of the subdivision corresponds to a **vertex** of the curve,
 - each **edge** of the subdivision corresponds to (and is perpendicular to) to an **edge** of the curve,
 - each **vertex** of the subdivision corresponds to a **linear region** in the complement of the curve.

![duality](duality.svg#center "An illustration of the duality between a tropical hypersurface and the subdivision of its Netwon polyotpe. Note that this visualization is partially misleading as it does not make too much sense to draw the curve and the subdivision in the same picture, as they live in different mathematical spaces.")

Although the subdivision retains some information about the coefficients, it also loses some. This means that we cannot deduce the exact position of the vertices of the curve simply looking at the subdivision. What we can say is how the curve will looks like *combinatorially*, which is which vertices are connected with which edges, which edges define which regions, and so forth. 

Something else we might notice is that there are two kind of linear regions in the complement of a tropical hypersurface: bounded and unbounded. The subdivision helps us to understand which ones are bounded and which ones are unbounded simply by looking at their corresponding vertices. In particular **bounded linear regions correspond to vertices in the interior of the Newton polytope, while unbounded regions correspond to vertices on the boundary of the Newton polytope**. In the following example one can verify that there are 

![duality2](duality2.svg#center "A tropical hypersurface and the corresponding subdivision of its Netwon polyotpe. Notice that there are six linear regions enclosed by the curve on the left and six interior vertices in the subdivision. Similarly there are fifteen unbounded linear regions and fifteen boundary points.")
<img src="\assets\img\2022-05-16-TropicalML\duality2.svg"  style="width:100%; display: block; margin-left: auto; margin-right: auto;" >

>Most of the tropical curves have been plotted with a [Python script](https://github.com/gabrieleballetti/tropical-curve-plotter) I have recently put together.
Source code for the other images:
[matplotlib](https://github.com/gabrieleballetti/unsolicited-tikz-pics/tree/main/matplotlib/TropicalML1).

