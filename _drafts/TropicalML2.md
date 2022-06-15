---
layout: post
title:  "Piecewise linear activations: Machine Learning meets Tropical Geometry (Part 2)"
---

In [the previous part]({% post_url 2022-05-16-TropicalML %}) we gave an introduction to Tropical Geometry. We spoke about tropical varieties, why it makes sense to define them the way they are defined, and how their Newton polytopes can be used to deduce information about their combinatorial structure. We now finally talk about how these tools can be used to get a better understanding of certain properties of neural network - such as their expressiveness - and how they vary when changing the structure of the network.

## Machine Learning meets Tropical Geometry

### A baby example

Let's start with the following minimal example. Con Consider a MLP with two input neurons, a hidden layer with three neurons and a single output neuron. This is not even as simple as it gets, as it will never be this simple

<img src="\assets\img\2022-06-20-TropicalML2\tiny_mlp.svg"  style="width:50%; display: block; margin-left: auto; margin-right: auto;" >