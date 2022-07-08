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

Let $$w_i$$ and $$b$$ be the weights and the bias at the output neuron. By inductive hypothesis we assume that each of the neurons at the layer $$L - 1$$ is given by a tropical rational function $$F^{(L-1)}_i(\mathbf{x}) \oslash G^{(L-1)}_i(\mathbf{x})$$. Then, the network defines the map

$$
\Phi(\mathbf{x}) = \max \left\{0, \sum_{i} w_i (F^{(L-1)}_i(\mathbf{x}) - G^{(L-1)}_i(\mathbf{x})) + b \right\}.
$$

At this point we can again manipulate the expression substracting all negative terms inside the max, and adding them outside as in

$$
\max \{ 0, a - b \} = \max \{ b , a \} - b
$$

where $$a$$ and $$b$$ are positive integers. This will allow us to ''remove all the $$-$$ from the $$\max$$'':

$$
\Phi(\mathbf{x}) = f(\mathbf{x}) - g(\mathbf{x})
$$

with

$$
\begin{align}
f(\mathbf{x}) &= \max\left\{ g(\mathbf{x}), h(\mathbf{x})\right\},\\
g(\mathbf{x}) &= \sum_{i | w_i < 0} (-w_i) F^{(L-1)}_i(\mathbf{x}) + \sum_{i | w_i > 0}  w_i G^{(L-1)}_i(\mathbf{x}),\\
h(\mathbf{x}) &= \sum_{i | w_i > 0} w_i F^{(L-1)}_i(\mathbf{x}) + \sum_{i | w_i < 0}  (-w_i) G^{(L-1)}_i(\mathbf{x}) + b.
\end{align}
$$

This is a bit intimidating, but what matters is that it is a difference of two expressions $$f(\mathbf{x})$$ and $$g(\mathbf{x})$$, and both these terms are sum, product and exponentiations of tropical polynomials and thus they are both tropical polynomials. This means that $$\Phi$$ is of the form

$$
\Phi(\mathbf{x}) = F(\mathbf{x}) \oslash G(\mathbf{x}),
$$

where $$F$$ and $$G$$ are tropical polynomials. **This proves that all MLPs are tropical rational maps.** Note that the fact that we fixed the output layer to consist of only one neuron is not restrictive, as this argument can be applied individually on all output neurons. Moreover this proof can be readapted to other piecewise linear activations (leaky ReLU, Maxout), other architectures (such as CNNs), and are not invalidated by pooling operations, residual blocks, concatenations, skip connections and so forth. As long as the activations are piecewise linear and all the other operations are composition of affine linear transformations, then this kind of argument can be used.

In [this paper by Zhang et al.](https://arxiv.org/abs/1805.07091) there is a proof of the statement above, and additionally they show that also the opposite statement is true, meaning that all tropical rational maps can be expressed as a ReLU neural network.

### Newton polytopes of ReLU networks

Once established that ReLU networks can be expressed as tropical rational maps, then we know that the decision boundaries of the network can be written as the union between the tropical variety of $$F$$ and the tropical variety of $$G$$. As discussed in the previous part, tropical varieties are dual objects to the Newton polytopes of their defining polynomials, so we can now shift our attention to Newton polytopes, and find out what $$\text{Newt}(F)$$ and $$\text{Newt}(G)$$ are. 

For the same network $$\Phi$$ as above ($$L$$ layers, the last of which is a single output neuron), we can deduce the Newton polytopes of $$F$$ and $$G$$ from those of the polynomials given from each of the neurons at the layer $$L - 1$$, which are $$\text{Newt}(F^{(L-1)}_i)$$ and $$\text{Newt}(G^{(L-1)}_i)$$, with $$i$$ indicating the $$i$$-th neuron on that layer. We do that using the explicit expression of $$\Phi(\mathbf{x})$$ we have previously found, obtaining:

$$\displaystyle{\text{Newt}(G) = \sum_{i | w_i < 0} (-w_i) \text{Newt}(F^{(L-1)}_i) + \sum_{i | w_i > 0}  w_i \text{Newt}(G^{(L-1)}_i)}
$$
$$\displaystyle{\text{Newt}(F) = \text{conv} \{ \text{Newt}(G) \cup \sum_{i | w_i > 0} w_i \text{Newt}(F^{(L-1)}_i) + \sum_{i | w_i < 0} (-w_i) \text{Newt}(G^{(L-1)}_i)} \cup \mathbf{0} \}.$$

Note that the sums above are Minkowski sum, in other words, the Newton polytope of $$G$$ is just a weighted Minkowski sum of the Newton polytopes $$\text{Newt}(F^{(L-1)}_i)$$ and $$\text{Newt}(G^{(L-1)}_i)$$,
while the Newton polytope of $$F$$ is the union of two such weighted Minkowski sums (and the origin). 

Note that - for all $$i$$ - $$\text{Newt}(F^{(1)}_i)$$ is a segment, while $$\text{Newt}(G^{(1)}_i)$$ is just a point (see the two layers network above as an example). At the next layer $$\text{Newt}(G^{(2)}_i)$$ is a *zonotope* (a Minkowski sum of edges), while $$\text{Newt}(F^{(2)}_i)$$ gets more complex, being the convex hull of the union of two zonotopes, plus the origin. At this point what is left is to count the *upper vertices* (the vertices of the upper hull) of these Minkowski sums, this will give us the number of linear regions we are looking for.

In [this paper by Montúfar et al.](https://arxiv.org/abs/2104.08135) the authors readapt a result by Weibel to count upper faces of Minkowski sums in terms of the number of upper faces of the Minkowski summands (Theorem 4.10). In the case of upper vertices for a Minkowski sum $$P = P_1 + \cdots + P_m$$ in $$\mathbb{R}^{n+1}$$, they obtain that they are at most

$$
\sum_{j = 0}^n(-1)^{n-j} { m - i - j \choose n-j} \sum_{S \in {[m] \choose j}} \prod_{i \in S} f_0(P_i)
$$

where $$f_0(P_i)$$ denotes the number of vertices of $$P_i$$. From this bound one can deduce that the maximum number of linear region of a shallow network with $$n$$ inputs and a single layer of ReLU units is

$$
\sum_{j = 0}^n {m \choose j},
$$

and that this bound is sharp (Theorem 3.6). Note that in the paper this is done in the more general settings of maxout units.

Iteratively applying this result to our deep networks setting - a network $$\Phi$$ ReLU activations, $$L$$ layers with $$n_1, \ldots, n_L$$ neurons - one can obtain the following asymptotic (and tight) bounds: the number of linear regions of $$\Phi$$ is (asymptotically) at least $$2^{L n_0}\prod_{l=1}^L n_l^{n_0}$$ and at most $$2^{2 L n_0} \prod_{l=1}^L n_l^{n_0}$$. This means that the number of linear regions grows exponentially both with the network depth and the layers width.

### An example, a tiny ReLU MLP

As an example we follow Example C.2 from [Zhang et al.](https://arxiv.org/abs/1805.07091), which has very nice pictures. Consider a MLP with two input neurons, a hidden layer with five neurons and a single output neuron, with ReLU activations at each non-input neuron.

<img src="\assets\img\2022-06-20-TropicalML2\tiny_mlp.svg"  style="width:50%; display: block; margin-left: auto; margin-right: auto;" >

Then this network defines a map $$\Phi$$ mapping the input $$(x_0, x_1)$$ as

$$
\Phi(x_1, x_2) = \max \left\{0,  \sum_{i = 1}^5  w^{(2)}_i \max \left\{0, w^{(1)}_{i,1} x_1 + w^{(1)}_{i,2} x_2 + b^{(1)}_i \right\} + b^{(2)}\right\}.
$$

for some weights and biases. We now fix some values for them:

$$
\mathbf{w}^{(1)} = 
\begin{pmatrix}
-1 &  1\\
 1 & -3\\
 1 &  2\\
-4 &  1\\
 3 &  2\\
\end{pmatrix}, \quad
\mathbf{b}^{(1)} = 
\begin{pmatrix}
1\\
-1\\
2\\
0\\
-2\\
\end{pmatrix}, \quad
\mathbf{w}^{(2)} = 
\begin{pmatrix}
1\\
2\\
1\\
-1\\
-3\\
\end{pmatrix}, \quad
b^{(2)}= 0.
$$

By applying the "negative-weights-moving" procedure we described earlier we can find an explicit expression for $$\Phi(x_0, x_1)$$:

$$
\Phi(x_0, x_1) = F(x_0, x_1) \oslash G(x_0, x_1)
$$

with

$$
\begin{align}
F(x_0, x_1) &= G(x_0, x_1) \oplus H(x_0, x_1),\\
G(x_0, x_1) &= (x_2 \oplus x_1^4) \odot ((-2) \odot x_1^3 x_2^2 \oplus 0)^3 \odot x_1 \odot (x_2^3)^2,\\
H(x_0, x_1) &= (1 \odot x_2 \oplus x_1) \odot ((-1) \odot x_1 \oplus x_2^3)^2 \odot (2 \odot x_1x_2^2 \oplus 0) \odot x_1^4.
\end{align}
$$

The Netwon polytope $$\text{Newt(G)}$$ of $$G$$ is

<img src="\assets\img\2022-06-20-TropicalML2\g.svg"  style="width:50%; display: block; margin-left: auto; margin-right: auto;" >,

the Netwon polytope $$\text{Newt(H)}$$ of $$H$$ is

<img src="\assets\img\2022-06-20-TropicalML2\h.svg"  style="width:50%; display: block; margin-left: auto; margin-right: auto;" >

while the Netwon polytope $$\text{Newt(F)}$$ of $$F$$ is

<img src="\assets\img\2022-06-20-TropicalML2\f.svg"  style="width:70%; display: block; margin-left: auto; margin-right: auto;" >

Note that $$\text{Newt(F)}$$ is the convex hull of the union of the previous two. These images are from the paper.