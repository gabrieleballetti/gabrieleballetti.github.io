---
layout: post
title:  "Piecewise linear activations: Machine Learning meets Tropical Geometry (Part 2)"
---

In [the previous part]({% post_url 2022-05-16-TropicalML %}) we gave an introduction to Tropical Geometry. We spoke about tropical varieties, why it makes sense to define them the way they are defined, and how their Newton polytopes can be used to deduce information about their combinatorial structure. We now finally talk about how these tools can be used to get a better understanding of certain properties of neural network - such as their expressiveness - and how they vary when changing the structure of the network.

## Tropical varieties of neural networks

### A tiny multilayer perceptron

Let's start with the following minimal example. Consider a MLP with two input neurons, a hidden layer with three neurons and a single output neuron, with some activation function $$\phi$$ at each non-input neuron.

<img src="\assets\img\2022-06-20-TropicalML2\tiny_mlp.svg"  style="width:50%; display: block; margin-left: auto; margin-right: auto;" >

Then this model defines a map $$\Phi$$ mapping the input $$\mathbf{x} = (x_0, x_1)$$ as

$$
\Phi (\mathbf{x}) = \phi(A^{(1)} \phi(A^{(0)} \mathbf{x} + \mathbf{b}^{(0)}) + \mathbf{b}^{(1)}),
$$

where $$A^{(0)} \in \mathbb{R}^{2 \times 3}$$ and $$\mathbf{b}^{(0)}  \in \mathbb{R}^{3}$$ represent the six weights and three biases for the hidden layer, and $$A^{(1)} \in \mathbb{R}^{3 \times 1}$$ and $$\mathbf{b}^{(1)} \in \mathbb{R}$$ are the three weights and single bias for the output layer. If we suppose that $$\phi$$ is a rectified linear unit, i.e. $$\phi(x) = \max(0, x)$$ then we can rewrite the map defined by the model as the following compact form

$$
\Phi (\mathbf{x}) = \max \{ 0, A^{(1)} \max \{ 0, A^{(0)} \mathbf{x} + \mathbf{b}^{(0)} \} + \mathbf{b}^{(1)} \}.
$$

Before we could interpret this map as a tropical polynomial (after all it is just a composition of the max operator and affine linear forms), we should be careful about the following subtlety. We want to be able to expand products such as 

$$
A \max \{ 0,\mathbf{x} \} = \max \{ 0, A \mathbf{x} \},
$$

but this is true only if the all the entries of $$A$$ are non-negative, which is clearly not the case. We get around this by decomposing $$A$$ into the difference of two nonnegative valued matrices

$$
A = A_+ - A_-
$$

by keeping all positive entries in $$A_+$$ and all negative entries in $$A_-$$, but in the latter we flip their signs. In this way the following holds:

$$
A \max \{ 0,\mathbf{x} \} = \max \{ A_- \mathbf{x}, A_+ \mathbf{x} \} - A_-\mathbf{x}.
$$

We can now use this to simplify the map defined by the tiny neural network above, which now (if I did the math right, which rarely happens) can be expressed as 

$$
\Phi (\mathbf{x}) = \max \{ A^{(1)}_- \mathbf{y} - \mathbf{b}^{(1)}, A^{(1)}_- \mathbf{y}, A^{(1)}_+ \mathbf{y} \} - A^{(1)}_- \mathbf{y} +  \mathbf{b}^{(1)}, \quad \text{ where } \mathbf{y} = A^{(0)} \mathbf{x} + \mathbf{b}^{(0)}.
$$

What is important here is that we have expressed the map defined by the neural network as the difference of maps which are now tropical maps:

$$
\Phi (\mathbf{x}) =  F(\mathbf{x}) - G((\mathbf{x})).
$$



which tropically corresponds to the so called *Freshman's dream*, i.e. the equality

$$
(x_0 \oplus x_1)^a = x_0^a \oplus x_1^a.
$$

But this is based on the assumption that $$a$$ is a non-negative integer, while all the weights of the expression above are real numbers without any restriction. Luckily passing from non-negative integer to non-negative real numbers is not a big deal. Traditionally Tropical Geometry is studies as a parallel to classical algebraic geometry, which deals with polynomials. Since polynomials have nonnegative integers appearing as exponents, then tropical mathematics is usually done with nonnegative integers as exponents. On the other hand, tropical maps, their tropical varieties and Newton polytopes and all the results about them we discussed in the previous post still make sense by using real numbers as exponents. While in the classical setting it might not be clear what $$(-1)^\pi$$ means, if interpreted tropically that expressions just means $$\pi \cdot (-1)$$.

This solve half of the problem, in the expression above we would like to expand $$A^{(1)} \max \{ 0, A^{(0)} \mathbf{x} + \mathbf{b}^{(0)} \} + \mathbf{b^{(1)}} $$
