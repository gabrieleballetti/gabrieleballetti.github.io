---
layout: post
title:  "Fun with string art"
---

String art is a technique to draw by weaving string between nails, obtaining a picture entirely composed of straight lines. Its origins are probably very old, but a computational approach to produce photorealistic results has been popularized by artist Petros Vrellis in 2016.

I have recently stumbled upon a [blog post](https://possiblywrong.wordpress.com/2022/01/22/string-art/) on Possibly Wrong about this topic, and I decided to give it a go myself. It turns out that even a simple algorithm can yield good results, especially if we allow many nails and a very long string.

<p align="center">
<img src="\assets\img\2022-08-04-string-art\ada_s.png"  style="width:40%; display: block; margin-left: auto; margin-right: auto;" ><br>
<em>String art portrait of Ada Lovelace with 256 nails, obtained with my algorithm.</em>
</p>

### A naïve algorithm

Once a grayscale input image is fixed, the first idea was to use a very simple greedy algorithm. Starting from any nail, we can look at all the possible lines from the starting nail to any of the others, and choose the *darkest* one, then wind the string around that nail and reiterate. By *darkest* we mean that we imagine this line overlapped to the original image, look at the average value of the pixels below the line, and pick the line with the lowest average. 
Let's fix some notation:
 - $$I_0(x)$$ is the brightness of the original image $$I_0$$ at its point $$x$$
 - $$\ell_{n_0,n_1}$$ is a line from the $$n_0$$-th nail to the $$n_1$$-th nail.

The goal it to find a sequence of nails $$n_0, n_1, \ldots, n_K$$ which creates the string image, stopping at some step $$K$$ which we need to find.

We decide to start with the nail $$n_0$$, and we look for the next nail $$n_1$$ by minimizing the quantity

$$\frac{\int_{x \in \ell_{n_0,n_1}} I_0(x) dx}{\text{length}(\ell_{n_0,n_1})}$$

over all possible choices of the second nail $$n_1 \neq n_0$$.

Now that the first step is done and we have stretched the string from $$n_0$$ to $$n_1$$, it would be tempting to simply find $$n_2$$ the exact same way. The first problem is that each nail as a uniquely determined successor, this means that if at some point we go back to $$n_0$$ (or any of the previous nails), then we end up in a loop without a possibility of adding new lines. This can be easily solved by keeping track of which lines has already been added. Then we need to figure out a way to stop, otherwise, since already added lines cannot be used again, we will simply exhaust all the possible lines until we hit a nail which is already connected to all the others. Intuitively, a good moment to stop is when the average brightness of the best line is at most the halfway point of the greyscale spectrum, as drawing a line exceeding that quantity will surely bring us further away from the input image than not drawing it. Here is what this algorithm produces, stopping at different points of the grayscale spectrum.

<p align="center">
<img src="\assets\img\2022-08-04-string-art\ada_naive.png"  style="width:90%; display: block; margin-left: auto; margin-right: auto;" ><br>
<em>String art portrait of Ada Lovelace with 256 nails, obtained with this naïve algorithm, stopping at different values of the grayscale spectrum.</em>
</p>

Note that some details are actually captured, like the shape of the face and the dark background, but we are far from creating a photorealistic picture. 

### A slightly refined algorithm

So far we have an algorithm which is good at drawing dark and bright regions of the canvas, but it misses all the details. The reason is that it focuses on the dark regions, keeping drawing dark lines regardless of what the canvas currently looks like. The idea is to use this information to better choose the following nail. So let's perform the fist step as described below, but once we found $$n_1$$ we modify the original input image $$I_0$$ by whitening the pixels below $$\ell_{n_0, n_1}$$, producing the image $$I_1$$. We do this at each step, producing intermediate images $$I_1, I_2, \ldots$$. At this point at the $$k$$-th step, we try to minimize the quantity

$$\frac{\int_{x \in \ell_{n_{k-1},n_k}} I_{k}(x) dx}{\text{length}(\ell_{n_{k-1},n_k})}$$

over all possible choices of $$n_k$$. In this way the algorithm dares drawing lines where we need some darker details, instead of focusing on ''safer'' dark areas. Note that we still need to stop the algorithm for some average value for the new lines, but this time we could be more conservative, and already for some low values on the grayscale spectrum we get some improved result.

<p align="center">
<img src="\assets\img\2022-08-04-string-art\ada_refined.png"  style="width:90%; display: block; margin-left: auto; margin-right: auto;" ><br>
<em>String art portrait of Ada Lovelace with 256 nails, obtained with the refined algorithm, stopping at different values of the grayscale spectrum.</em>
</p>

This is already an improvement from the previous attempt, as we can see some details as the eyes and the hair, but it is still barely recognizable.

### The final algorithm

Note how the previous algorithm sketches some details, like the eyes, but then does not define them properly. The reason seems to be that the intermediate image $$I_k$$ loses information about these details too fast, as they quickly get whitened by the previous lines. The next step is then to be more conservative with this whitening process, and instead of subtracting a black line from $$I_k$$ to obtain $$I_{k+1}$$, we subtract a gray one with some opacity. Opacity works multiplicatively, intuitively this means that initially the lines will brighten dark pixels fast, but then the same pixels will take longer and longer to be turned into white ones. On the canvas this has the effect of forcing more lines to overlap on darker spots. This worked like a charm, here are some results for different values of opacity.

<p align="center">
<img src="\assets\img\2022-08-04-string-art\ada_final.png"  style="width:90%; display: block; margin-left: auto; margin-right: auto;" ><br>
<em>String art portrait of Ada Lovelace with 256 nails, obtained with the final algorithm, for different opacity values.</em>
</p>

For the picture at the top of the page I have picked a value of $$0.15$$.

### Is the greedy approach restrictive?

One might argue that this greedy approach might be too rough, what if instead of picking the best line, we pick one which is slightly worse but which allows us to draw better lines later on? This makes absolutely sense, and such an algorithm would surely improve the final result. We could also derive from the previous one. Instead of minimizing the brightness of a line from a given nail over all the possible choices for the following nail, we could at each step minimize the same quantity over all the possible choices of both nails. In this way we could create an image made up of disconnected segments on the canvas, which then we can attempt to reorder to see if they can be concatenated correctly. This is extremely unlikely to be the case, this problem is equivalent to find an [Eulerian path](https://en.wikipedia.org/wiki/Eulerian_path) in the graph defined by all the segments, and a necessary (and sufficient) condition for one to exist is that this graph is connected (extremely likely if there are many nails) and every vertex has even degree (extremely unlikely if there are very nails). But at this point we can force the graph to satisfy this condition by progressively adding lines until all vertices have even degree, which can be done with little impact on the final image. 

The question is how much better the results will get by doing so? We can make an experiment comparing the result of the greedy algorithm with a modified "disconnected" one drawing the best line at each step, without bothering checking that the latter can be realized with a unique connected string. 

<p align="center">
<img src="\assets\img\2022-08-04-string-art\ada_comparison.png"  style="width:70%; display: block; margin-left: auto; margin-right: auto;" ><br>
<em>String art portrait of Ada Lovelace with 256 nails. Comparison between the results of the greedy algorithm (left) and the modified "disconnected" version picking the best line at each step (right). </em>
</p>

While differences are expected when the number of nails is small, the results are almost identical when the number grows. Since for photorealistic applications the number of nails needs to be in the hundreds, then I decided to stick with the greedy algorithm (which is also way faster).

### Final touches

At this point I was quite satisfied with the result. As a final touch I felt that it would be against the spirit of the string artist to stretch short lines between neighboring nails, so I added the options to control how close two consecutive nails can be. For the picture at the top of the page the number I have picked is $$32$$, one eight of the total number of nails, meaning that two consecutive nails on the string path are at least $$32$$ nails away from each other.

<p align="center">
<img src="\assets\img\2022-08-04-string-art\ada_neighbors.png"  style="width:90%; display: block; margin-left: auto; margin-right: auto;" ><br>
<em>String art portrait of Ada Lovelace with 256 nails, obtained with the final algorithm, for different values of minimum distance between two nails</em>
</p>

Note that this choice has the effect of discoloring outside a circle, while keeping most of the details inside of it. Such circle is the one tangent to all the chords between the closest possible nails. 

### Results

