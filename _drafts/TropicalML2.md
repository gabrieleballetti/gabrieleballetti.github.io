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

Before we could interpret this map tropically, we need to remark that **we allow real numbers as (tropical) exponents**. Since polynomials are traditionally studied with nonnegative integers appearing as exponents, Tropical Geometry is also generally introduced with nonnegative integers as exponents. On the other hand, tropical maps, their tropical varieties and Newton polytopes and all the results about them we discussed in the previous post still make sense, and all their properties still hold, when using real numbers as exponents. This is due to the ``linearity'' of Tropical Geometry. While in the classical setting it might not be clear what $$(-1)^\pi$$ means, if interpreted tropically that expressions just means

$$
(-1)^{\odot \pi} = \pi \cdot (-1) = -\pi.
$$

In order to simplify the expression above, we need to be careful that $$a \cdot \max\{ x_0, x_1 \}$$ is not just $$\max\{ ax_0, ax_1 \}$$, unless $$a \geq 0$$. We get around this by splitting the weights in positive and negative. In compact notation, we decompose a weight matrix $$A$$ into the difference of two nonnegative valued matrices

$$
A = A_+ - A_-
$$

by keeping all positive entries in $$A_+$$ and all negative entries in $$A_-$$, but in the latter we flip their signs. In this way the following holds:

$$
A \max \{ 0,\mathbf{x} \} = \max \{ A_- \mathbf{x}, A_+ \mathbf{x} \} - A_-\mathbf{x}.
$$

We can simplify the map defined by the tiny neural network above, which now (if I did the math right) should look like

$$
\Phi (\mathbf{x}) = \max \{ A^{(1)}_- \mathbf{y} - \mathbf{b}^{(1)}, A^{(1)}_- \mathbf{y}, A^{(1)}_+ \mathbf{y} \} - A^{(1)}_- \mathbf{y} +  \mathbf{b}^{(1)}, \quad \text{ where } \mathbf{y} = A^{(0)} \mathbf{x} + \mathbf{b}^{(0)}.
$$

Without explicitly expressing $$\Phi$$ in terms of $$x_0$$ and $$x_1$$, we can note that the first half is a maximum among affine linear forms, to which another affine linear form is subtracted. Now this can be interpreted tropically, as the first part is just a tropical sum of tropical polynomials, and hence a tropical polynomial, and the latter is just a tropical polynomial. If we introduce the symbol $$\oslash$$ to denote the inverse of the tropical multiplication (i.e. the classical subtraction) then **$$\Phi$$ is a tropical rational map**:

$$
\Phi (\mathbf{x}) =  F(\mathbf{x}) \oslash G(\mathbf{x}).
$$

