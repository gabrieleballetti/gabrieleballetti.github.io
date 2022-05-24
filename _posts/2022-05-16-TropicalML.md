---
layout: post
title:  "Piecewise linear activations: Machine Learning meets Tropical Geometry (Part 1)"
---

<img src="\assets\img\2022-05-16-TropicalML\quartic_surf.svg"  style="width:40%; display: block; margin-left: auto; margin-right: auto;" >

> What tropical surfaces look like (a tropical quartic surface, from [this paper](https://arxiv.org/abs/1806.02236) )

Activation functions decide what is to be fired to the following neuron in a neural network depending on the input signal received. Since they generally are the only nonlinear component in a model architecture, they are what truly allow networks to learn complex patterns, as without them network would only approximate linear functions (and most of their structure would be redundant). Although traditionally the sigmoid was the most common choice for an activation function, in modern neural network the default recommendation is to use the Rectified Linear Unit (ReLU) defined as $$x \mapsto \max(0,x)$$, or its more flexible version Leaky ReLU $$x \mapsto \max(ax,x)$$.

<img src="\assets\img\2022-05-16-TropicalML\activation_functions.svg"  style="width:100%; display: block; margin-left: auto; margin-right: auto;" >

Using a piecewise linear activation function makes the network a piecewise linear function, i.e. the input space gets partitioned in polyhedral regions, and in each region the network behaves linearly. 

Studying such linear regions is an important step in the understanding of deep neural networks: since their goal is to be good approximators, their number of linear regions is a good measure of their *expressiveness*. In recent years a lot of progress to find lower and upper bounds for this quantities has been made (with the best bounds so far found by [Serra et al.](https://arxiv.org/abs/1711.02114), according to [this recent paper by Trimmel et al.](https://openreview.net/forum?id=IqtonxWI0V3)). Note that the number of regions is massive: lower bounds tells us that there are at least $$10^{80}$$ even for small architectures. If you compare this number with the general number of training samples ($$\leq 10^6$$), it is clear that **basically none of the linear regions contains training data**!

> This raises the question of how representative the number of linear regions is for network performance and how information extracted from training samples passes on to the many linear regions free of data for successful generalization to test data (from [Trimmel et al.](https://openreview.net/forum?id=IqtonxWI0V3)).

An important tool for this kind of investigations has been *tropical geometry*: it was recently shown ([Charisopoulos & Maragos (2018)](https://arxiv.org/abs/1805.08749v2), [Zhang et al. (2018)](https://arxiv.org/abs/1805.07091)) that **ReLU neural network functions are the same as tropical rational maps**. 

I am going to present this connection in a series of blog posts. In this first one I focus on the math part, introducing the main ideas behind tropical geometry.

## A gentle introduction to Tropical Geometry
Tropical Geometry is a relatively recent field of study in mathematics - laying somewhere between algebraic geometry and combinatorics - where the traditional study of algebraic varieties is substituted by the study of some polyhedral objects, defined by interpreting the polynomials over the *tropical semiring*. The best reference for tropical geometry is the book *Introduction to Tropical Geometry* by Diane Maclagan and Bernd Sturmfels. In it you will find the slogan "a tropical variety is a *combinatorial shadow* of the original variety", which is now is repeated as a mantra in paper and conferences (1870 results on Google for ["combinatorial shadow of"](https://www.google.com/search?q=%22combinatorial+shadow+of%22), probably all from this context). Remember to use it if you write a paper on this subject.

Another curiosity is the origin of the term *tropical* in Tropical Geometry, from the aforementioned book:

> The adjective “tropical” was coined by French mathematicians, notably Jean-Eric Pin, to honor their Brazilian colleague Imre Simon, who pioneered the use of min-plus algebra in optimization theory. There is no deeper meaning in the adjective “tropical”. It simply stands for the French view of Brazil. 

### Basic definitions

The *tropical semiring* is defined as $$\mathbb{T} := (\mathbb{R} \cup \{ -\infty \}, \oplus, \odot) $$ where

 - $$x \oplus y := max\{ x, y \}$$ is the *tropical addition*,
 - $$x \odot y := x + y$$ is the *tropical multiplication*.

The extra element $$-\infty$$ has been added to ensure that $$\mathbb{T}$$ has an additive identity, i.e. $$x \oplus -\infty = x$$, while $$0$$ is the multiplicative identity. This definition ensures associativity, commutativity and distributivity, although the addition is not invertible, for example $$x \oplus 1 = 0$$ has no solutions. For this reason $$\mathbb{T}$$ is a semiring, not a ring. Note that $$\min$$ tends to be used more frequently instead of $$\max$$ by mathematicians, but since it is only a matter of conventions we are going to use the $$\max$$-convention, as it will adapt better to the machine learning context. In any case, the whole theory is equivalent regardless of this choice.

### Roots of tropical polynomials

If interpreted as functions over the tropical semiring, polynomials are piecewise linear functions, where the "pieces" - or *linear regions* - are polyhedral. Consider the univariate tropical polynomial

$$p(x) = x^2 \oplus 2 \odot x \oplus 1 = \max \{ 2x, x + 2, 1 \}$$,

in this case the linear regions are the intervals $$(-\infty,-1], [-1,2], [2,+\infty)$$. Here the values $$-1$$ and $$2$$ play a special role, as each of them belongs to two intervals. Moreover, they appear explicitly if we rewrite $$p$$ as $$(x \oplus -1) \odot (x \oplus 2)$$. It makes then sense to call them *tropical roots* of $$p$$, in analogy with what happens in the traditional settings.

Note that, as in the example below, the factorizations (and therefore the roots) in the tropical and classical settings are different.

<img src="\assets\img\2022-05-16-TropicalML\polynomial_factorization.svg"  style="width:100%; display: block; margin-left: auto; margin-right: auto;" >

The fact that a tropical polynomial can be factored as in the example above is a general property of tropical univariate polynomial functions, which means that the so-called *Fundamental Theorem of Algebra* holds tropically. Note the word "polynomial *function*". Indeed different polynomials define the same functions, just because some summands may play no role. For example, in the polynomial function defined by the polynomial

$$q_a(x) = x^2 \oplus a \odot x \oplus 0 = \max \{ 2x, x + a, 0 \}$$

the term $$a \odot x$$ plays no role whenever $$a \leq 0$$. In particular, even being different as polynomials (because they have different coefficients), $$q_{-1}(x)$$ and $$q_0(x)$$ define the same polynomial function $$x \mapsto \max(2x, 0)$$. So the correct way to state the Tropical Fundamental Theorem of Algebra is that *every tropical univariate polynomial can be replaced by an equivalent polynomial (defining the same polynomial function) that can be factored - in a unique way - into linear factors*.

To summarize, for univariate tropical polynomials, the roots are those elements in the domain belonging at least to two linear regions (or, equivalently, for which the maximum is attained at least twice) and, to confirm that this definition is meaningful, they appear in the "unique" factorization of the polynomial. 

### Tropical curves, surfaces and hypersurfaces

But what happens in higher dimensions, when we have more variables other than just $$x$$? In the classical setting, the concept of roots is extended as the *vanishing locus* or the *zeroes* of a polynomial, i.e. all the points in the domain where the polynomial evaluate to zero. These sets are called *algebraic curves* in $$\mathbb{R}^2$$ (such as an ellipse) for bivariate polynomials, *algebraic surfaces* $$\mathbb{R}^3$$ (such as a sphere) for trivariate polynomials, or - more generally - *algebraic hypersurfaces* when the dimension is higher or non specified. In the tropical setting the concept of tropical roots is extended by focusing on the linear regions: *given a polynomial, the algebraic tropical curve/surface/hypersurface defined by the polynomial is the set of all the points belonging to at least two linear regions*. 

Here is an example of how (the function defined by) a degree two tropical polynomial in two variables looks like.

<img src="\assets\img\2022-05-16-TropicalML\tropical_polynomial.svg"  style="width:80%; display: block; margin-left: auto; margin-right: auto;" >

And here is the algebraic tropical *quadric* (curve of degree two) it defines. In a way, the tropical curve is what you see when you look at tropical function "from above".

<img src="\assets\img\2022-05-16-TropicalML\tropical_curve.svg"  style="width:60%; display: block; margin-left: auto; margin-right: auto;" >

As we increase the degree of the polynomials, curves get more complex. Here is what a degree fifteen tropical curve with random integer coefficients looks like (I made a [neat script](https://github.com/gabrieleballetti/tropical-curve-plotter) to plot such curves).

<img src="\assets\img\2022-05-16-TropicalML\tropical_curve_random.svg"  style="width:60%; display: block; margin-left: auto; margin-right: auto;" >

There are several clues that confirm that with these definitions we are onto something. Every mathematician's favorite example is the tropical version of *Bézout's Theorem*. In the classical settings it states that two algebraic curves defined by polynomials of degrees $$c$$ and $$d$$ intersects in $$cd$$ points. Here the result is loosely stated, as it holds for "generic" curves defined on an algebraically closed field, and one need a notion of multiplicity when there is tangency. Usually it is stated for projective curves, as it takes a cleaner form. Anyway it should not sound too weird: two lines intersect in one point, a line and an ellipse intersect in two (possibly complex) points, two ellipses intersect in four (possibly complex) points, and so forth. The very same theorem holds tropically, and - if possible - it holds in a cleaner way, meaning that we do not need extra hypotheses such as working with complex numbers, or assuming that the curves have been chosen "generically" enough. All we need a concept of *stable* intersection to handle the cases where two curves are aligned (if you want to know more, have a look to Corollary 1.3.4 in the referenced book). In the example below we can see a line (orange, a degree one algebraic curve) intersecting a cubic (blue, a degree three algebraic curve) in three points, both in a classical and tropical environment.

<img src="\assets\img\2022-05-16-TropicalML\bezout.svg"  style="width:100%; display: block; margin-left: auto; margin-right: auto;" >
