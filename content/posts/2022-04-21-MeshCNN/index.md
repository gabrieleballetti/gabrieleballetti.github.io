---
title:  "MeshCNN: a convolutional neural network for meshes"
date: 2022-04-21T21:21:45+01:00
tags: ["machine learning", "geometry", "mesh", "segmentation"]
math: true
---

![head pic](T252.png#center)

> This post is an adaptation of my notes for a presentation I have recently given. All pictures are from the authors' [paper presentation page](https://ranahanocka.github.io/MeshCNN/).

Some time ago, while working on a mesh segmentation problem I found a very interesting paper which was presented at SIGGRAPH 2019 called [MeshCNN: A Network with an Edge](https://ranahanocka.github.io/MeshCNN/). The main result of the paper is to define convolution and (un)pooling operations on triangular meshes with the goal of allowing CNN techniques while preserving the connectivity information that would be lost if the 3D object was represented for example as a point cloud or with voxelization. These operations are moreover invariant to rotations, translations and uniform scale.

This concept reminded me of [this paper](https://arxiv.org/pdf/2109.09602.pdf) written - among others - by a coauthor of mine, where the authors train models to predict standard properties of polytopes, motivated by pure mathematics questions. In this paper the authors mostly use non-convolutional techniques such as MLPs (multilayer perceptron), and they make the input data invariant to rotation, translation and uniform scale by using [Plücker coordinates](https://en.wikipedia.org/wiki/Pl%C3%BCcker_coordinates)[^plucker] instead of standard point coordinates (actually they also use point coordinates, but they also notice that they yield worse results).

# The advantages of MeshCNN
In order to apply ML techniques for 3D data analysis various strategies have been used. Some of the most common are:
 - **multi-view 2D projections**, basically multiple "pictures" of the 3D object from multiple angles, on these one can apply CNN techniques;
 - **volumetric/voxelization**, i.e. storing the object data in a 3D occupancy grid, on which 3D analogues of standard CNN techniques can be applied;
 - **point clouds**, where either non-convolutional fully connected layers must be used (to ensure point-ordering invariance), or other local neighborhoods information must be computed in order to allow some sort of convolutional operations.

All these approaches share a common drawback: in most of real world application (maybe biased by the fact that I am working in the gaming industry), the starting object is often represented as a mesh, where connectivity information is already given by the mesh graph and then lost when passing to images, voxelization or point cloud. The goal of MeshCNN is to hold onto this existing information, and rather define an entirely new way to do convolutions directly on the meshes.

# Mesh convolution and (un)pooling
In MeshCNN a convolution is calculated for each edge, where and the input is a five dimensional vector. This vector represents basic geometry features for that edge, namely the dihedral angle, two inner angles and two edge-length ratios for each face. Note how these properties are invariant

![head pic](input_edge_features2.png#center)

In a sense if classic CNN is performed on image, using RGB color as an input vector for each pixel, in MeshCNN the mesh is the image, the edges are the pixels, and their geometric feature vectors are their RGB colors.
Since each edge on the mesh belongs exactly to two faces, each of which contributing with two additional edges, this naturally defines a fixed-sized convolutional neighborhood of four edges for all edges of the mesh, without having to worry about the boundary and padding (like we do with images). 

![head pic](mesh_conv.png#center)

At this point we need a pooling operation for downsampling the number of features in the network. To do so, the authors define a "mesh pooling" operation as an edge-collapse on the learned edge features, generating new mesh connectivity for the subsequent convolutions. For fully-convolutional tasks (such as segmentation) where localization matters, a mesh unpooling operation is used to restore the original mesh resolution. 

![head pic](mesh_pool_unpool.png#center)

The order in which this edge-collapse order is defined is given by the magnitude of the edge features, allowing the model to select
which regions of the mesh to collapse ad which to keep, depending on the task.

![head pic](T76.png#center)

# Results and remarks

The results obtained by model using MeshCNN look great both for classification and segmentation problems, improving several previous results. The authors also provided these great images that show how the mesh simplification process preserves the segmented parts of the initial mesh.

![](coseg_chair.png#center)

A final question left in my mind after reading the paper is about the choice of the geometric feature vector. Angles and edge ratios preserve rotation, translation and uniform scale, but the presence of angles implies that the vector will not be invariant up to *shearing operations*, i.e. general transformations by invertible unit matrices. This makes MeshCNN as-it-is not ideal for applications for the study of polytopes like the one mentioned earlier. Is there a completely $\mathrm{SL}_3(\mathbb{R})$-invariant vector which could be chosen?


[^plucker]: The authors remark that there are non-equivalent polytopes sharing the same Plücker coordinates, which one would be able to discriminate by adding an extra piece of combinatorial data called the quotient gradings, but do not investigate this further in the paper.