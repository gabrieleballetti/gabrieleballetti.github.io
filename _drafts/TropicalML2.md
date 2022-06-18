---
layout: post
title:  "Piecewise linear activations: Machine Learning meets Tropical Geometry (Part 2)"
---

In [the previous part]({% post_url 2022-05-16-TropicalML %}) we gave an introduction to Tropical Geometry. We spoke about tropical varieties, why it makes sense to define them the way they are defined, and how their Newton polytopes can be used to deduce information about their combinatorial structure. We now finally talk about how these tools can be used to get a better understanding of certain properties of neural network - such as their expressiveness - and how they vary when changing the structure of the network.

## Machine Learning meets Tropical Geometry

### A tiny multilayer perceptron

Let's start with the following minimal example. Consider a MLP with two input neurons, a hidden layer with three neurons and a single output neuron, with some activation function $$\phi$$ at each non-input neuron.

<img src="\assets\img\2022-06-20-TropicalML2\tiny_mlp.svg"  style="width:50%; display: block; margin-left: auto; margin-right: auto;" >

Then this model defines a map

$$
(x_0, x_1) = \mathbf{x} \mapsto \phi(A^{(1)} \phi(A^{(0)} \mathbf{x} + \mathbf{b}^{(0)}) + \mathbf{b^{(1)}}),
$$

where $$A^{(0)} \in \mathbb{R}^{2 \times 3}$$ and $$\mathbf{b}^{(0)}  \in \mathbb{R}^{3}$$ represent the six weights and three biases for the hidden layer, and $$A^{(1)} \in \mathbb{R}^{3 \times 1}$$ and $$\mathbf{b}^{(1)} \in \mathbb{R}$$ are the three weights and single bias for the output layer. If we suppose that $$\phi$$ is a rectified linear unit, i.e. $$\phi(x) = \max(0, x)$$ then we can expand the map defined by the model as

$$
(x_0, x_1) \mapsto \max(0, a^{(1)}_0 y_0 + a^{(1)}_1 y_1 + a^{(1)}_{2} y_2 + b^{(1)})
$$

with

$$
y_i = \max (0, a^{(0)}_{0,i} x_0 + a^{(0)}_{1,i} x_1 + b^{(0)}), \quad \text{ for } i = 1,2,3.
$$

At this point it would be tempting to interpret this map as a tropical polynomial, after all it is just a composition of $$\max$$ and affine linear forms. In order to do so, we should be careful about the following subtlety. We want to be able to perform simplifications such as $$ a \cdot \max(x_0, x_1) = \max(ax_0, ax_1)$$. Tropically this holds, it's the so called *Freshman's dream*, i.e. the equality

$$
(x_0 \oplus x_1)^a = x_0^a \oplus x_1^a.
$$

But this is based on the assumption that $$a$$ is a non-negative integer, while all the weights of the expression above are real numbers without any restriction. Luckily passing from non-negative integer to non-negative real numbers is not a big deal. Traditionally Tropical Geometry is studies as a parallel to classical algebraic geometry, which deals with polynomials. Since polynomials have nonnegative integers appearing as exponents, then tropical mathematics is usually done with nonnegative integers as exponents. **On the other hand, tropical maps, their tropical varieties and Newton polytopes and all the results about them we discussed in the previous post still make sense by using real numbers as exponents**. While in the classical setting it might not be clear what $$(-1)^\pi$$ means, if interpreted tropically that expressions just means $$\pi \cdot (-1)$$.

This solve half of the problem, as $$a \cdot \max(x_0, x_1)$$ is still not equal $$\max(ax_0, ax_1)$$ whenever $$a < 0$$. To solve this we will need a smarter trick.
