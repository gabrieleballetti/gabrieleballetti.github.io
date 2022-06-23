---
layout: post
title:  "Piecewise linear activations: Machine Learning meets Tropical Geometry (Part 2)"
---

In [the previous part]({% post_url 2022-05-16-TropicalML %}) we gave an introduction to Tropical Geometry. We spoke about tropical varieties, why it makes sense to define them the way they are defined, and how their Newton polytopes can be used to deduce information about their combinatorial structure. We now talk about how these tools can be used to get a better understanding of decision boundaries of neural network and how they are influenced by the structure of the network.

### ReLU Neural Networks are tropical rational maps

Before we could interpret this map tropically, we need to remark that from now on **we allow real numbers as (tropical) exponents** in tropical polynomials. Since polynomials are traditionally studied with nonnegative integer exponents, Tropical Geometry is also generally introduced with nonnegative integers as exponents. On the other hand, tropical maps, their tropical varieties and Newton polytopes and all the results about them we discussed in the previous post still hold, and all their properties still hold, when using real numbers as exponents. This is due to the ``linearity'' of Tropical Geometry. While in the classical setting it might not be even clear what $$(-1)^\pi$$ means, if interpreted tropically that expressions just means

$$
(-1)^{\odot \pi} = \pi \cdot (-1) = -\pi.
$$

The reason we loosen our definitions is that weights of neural networks will act as tropical exponents, and weights are real numbers. Consider an hypothetical one-layer MLP with only neurons in the input layer and a ReLU activation in the output neuron.

<img src="\assets\img\2022-06-20-TropicalML2\one_layer_mlp.svg"  style="width:33%; display: block; margin-left: auto; margin-right: auto;" >

Suppose that the weights have values $$2$$ and $$-3$$ and the bias at the output neuron has value $$1$$. Then the network defines the following map from the input space to the output space.

$$
\Phi(x_0, x_1) = \max \{0, 2 x_0 - 3 x_1 + 1 \}.
$$

Written as it is it is not obvious hot to interpret $$\Phi$$ tropically, but with some simple manipulation we can rewrite it as

$$
\Phi(x_0, x_1) = \max \{3 x_1, 2 x_0 + 1 \} - 3 x_1.
$$

At this point we can clearly see two tropical polynomials: the max becomes a sum of two monomials, and hence a tropical polynomial, and the latter is just a tropical monomial. If we introduce the symbol $$\oslash$$ to denote the inverse of the tropical multiplication (i.e. the classical subtraction) then $$\Phi$$ is a tropical rational map:

$$
\Phi(x_0, x_1) =  (1 \odot x_0^2 \oplus x_1^3) \oslash x_1^3.
$$

This is of course a minimal example, but can be used as the base case for inductively showing that any ReLU MLP is a tropical rational map. It is easy to see that the same argument holds for any number of input neurons.

For the inductive step consider a larger ReLU MLP with $$L$$ layers with the last one consisting of only one output neuron.

<img src="\assets\img\2022-06-20-TropicalML2\mlp.svg"  style="width:90%; display: block; margin-left: auto; margin-right: auto;" >

Let $$w_i$$ and $$b$$ be the weights and the bias at the output neuron. By inductive hypothesis we assume that each of the neurons at the layer $$L - 1$$ is given by a tropical rational function $$F_i(\mathbf{x}) \oslash G_i(\mathbf{x})$$. Then, the network defines the map

$$
\Phi(\mathbf{x}) = \max \left\{0, \sum_{i} w_i (F_i(\mathbf{x}) - G_i(\mathbf{x})) + b \right\}
$$

At this point we can again manipulate the expression using the property substracting all negative terms inside the max, and adding them outside.

$$
\scriptsize
\Phi(\mathbf{x}) = \max \left\{\sum_{i | w_i < 0} (-w_i) F_i(\mathbf{x}) + \sum_{i | w_i \geq 0}  w_i G_i(\mathbf{x}), \sum_{i | w_i \geq 0} w_i F_i(\mathbf{x}) + \sum_{i | w_i < 0}  (-w_i) G_i(\mathbf{x}) + b \right\} - \left( \sum_{i | w_i < 0} (-w_i) F_i(\mathbf{x}) + \sum_{i | w_i \geq 0}  w_i G_i(\mathbf{x}) \right).
$$

This is a bit intimidating, but what matters is that it is a difference of two terms, and both these terms are sum, product and exponentiations of tropical polynomials and thus they are both tropical polynomials. This means that $$\Phi$$ is of the form

$$
\Phi(\mathbf{x}) = F(\mathbf{x}) \oslash G(\mathbf{x}),
$$

where $$F$$ and $$G$$ are tropical polynomials. **This proves that all MLPs are tropical rational maps.** Note that the fact that we fixed the output layer to consist of only one neuron is not restrictive, as this argument can be applied individually on all output neurons. Moreover this proof can be readapted to other piecewise linear activations (leaky ReLU, Maxout), other architectures (such as CNNs), and are not invalidated by pooling operations, residual blocks, concatenations, skip connections and so forth. As long as the activations are piecewise linear and all the other operations are composition of affine linear transformations, then this kind of argument can be used.

In [this paper by Zhang et al.](https://arxiv.org/abs/1805.07091) there is a  formal proof of the statement above, and additionally they show that also the opposite statement is true, meaning that all tropical rational maps can be expressed as a ReLU neural network.

### An example, a tiny multilayer perceptron

Let's be a bit more explicit with another small network. Consider a MLP with two input neurons, a hidden layer with three neurons and a single output neuron, with some activation function $$\phi$$ at each non-input neuron.

<img src="\assets\img\2022-06-20-TropicalML2\tiny_mlp.svg"  style="width:50%; display: block; margin-left: auto; margin-right: auto;" >

Then this model defines a map $$\Phi$$ mapping the input $$\mathbf{x} = (x_0, x_1)$$ as

$$
\Phi (\mathbf{x}) = \phi(A^{(1)} \phi(A^{(0)} \mathbf{x} + \mathbf{b}^{(0)}) + \mathbf{b}^{(1)}),
$$
