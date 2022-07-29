---
layout: post
title:  "Fun with string art"
post_image: '<img src="\assets\img\2022-07-29-string-art\aec.png"  style="width:60%; display: block; margin-left: auto; margin-right: auto;" >'
---

String art is a technique to draw by weaving a string between pins, obtaining a picture entirely composed of straight lines. Its origins are probably ancient, but a computational approach to produce photorealistic results has been popularized by artist Petros Vrellis in 2016.

I have recently stumbled upon a [blog post](https://possiblywrong.wordpress.com/2022/01/22/string-art/) by Possibly Wrong about this topic, and I decided to give it a go myself, before looking at any other implementation. It turns out that even a simple algorithm can yield good results, especially if we allow many pins and a very long string.

<p align="center">
<img src="\assets\img\2022-07-29-string-art\ada.gif"  style="width:40%;" >
<img src="\assets\img\2022-07-29-string-art\ada_s.png"  style="width:40%;" ><br>
<em>String art portrait of Ada Lovelace with 256 pins, obtained with my implementation (<a href="https://github.com/gabrieleballetti/string-art">GitHub</a>).<br> A frame every 50 lines (left) and the final result (right).</em>
</p>

### First attempt

Once a grayscale input image is fixed, the first idea was to use a very simple greedy algorithm. Starting from any pin, we can look at all the possible lines from the starting pin to any of the others, and choose the *darkest* one, then wind the string around that pin and reiterate. By *darkest* we mean that we imagine this line overlapped to the original image, look at the average value of the pixels below the line, and pick the line with the lowest average. 
If
 - $$I_0(x)$$ is the brightness of the original image $$I_0$$ at its point $$x$$ (lower means darker),
 - $$\ell_{p_0,p_1}$$ is a line from the $$p_0$$-th pin to the $$p_1$$-th pin,

then the goal it to find a sequence of pins $$p_0, p_1, \ldots, p_K$$ which creates the string image, stopping at some step $$K$$ which we need to find.

We decide to start with any pin $$p_0$$, and we look for the next pin $$p_1$$ by minimizing the quantity

$$f_{I_0,p_0}(n) = \frac{\int_{x \in \ell_{p_0,n}} I_0(x) dx}{\text{length}(\ell_{p_0,n})}$$

over all possible choices of the second pin $$n$$.

Now that the first step is done and we have stretched the string from $$p_0$$ to $$p_1$$, it would be tempting to simply find $$p_2$$ the exact same way. The first problem is that each pin as a uniquely determined successor, this means that if at some point we go back to $$p_0$$ (or any of the previous pins), then we end up in a loop without a possibility of adding new lines. This can be easily solved by keeping track of which lines has already been added. Then we need to figure out a way to stop, otherwise, since already added lines cannot be used again, we will simply exhaust all the possible lines until we hit a pin which is already connected to all the others. Intuitively, a good moment to stop is when the average brightness of the best line is at most the halfway point of the greyscale spectrum, as drawing a line exceeding that quantity will surely bring us further away from the input image than not drawing it. Here is what this algorithm produces, stopping at different points of the grayscale spectrum.

<p align="center">
<img src="\assets\img\2022-07-29-string-art\ada_naive.png"  style="width:90%; display: block; margin-left: auto; margin-right: auto;" ><br>
<em>The portrait of Ada Lovelace with 256 pins obtained with this naïve algorithm, stopping at different values of the grayscale spectrum.</em>
</p>

Note that some details are actually captured, like the shape of the face and the dark background, but we are far from creating a photorealistic picture. 

### A slightly refined algorithm

So far we have an algorithm which is good at drawing dark and bright regions of the canvas, but it misses all the details. The reason is that it focuses on the dark regions, keeping drawing dark lines regardless of what the canvas currently looks like. The idea is to use this information to better choose the following pin. So let's perform the fist step as described below, but once we found $$p_1$$ we modify the original input image $$I_0$$ by whitening the pixels below $$\ell_{p_0, p_1}$$, producing the image $$I_1$$. We do this at each step, producing intermediate images $$I_1, I_2, \ldots$$, intuitively we can think of this as

$$I_k = I_{k-1} - \ell_{p_{k-1}, p_k}, \text{ for } k > 0,$$

where the subtraction indicates that the dark pixels of the line are subtracted from the pixels of the image, making them brighter. At the $$k$$-th step, we find $$p_k$$ as the pin $$n$$ minimizing the quantity

$$f_{I_k,p_k}(n) = \frac{\int_{x \in \ell_{p_{k},n}} I_{k}(x) dx}{\text{length}(\ell_{p_{k},n})}.$$

In this way the algorithm dares drawing lines where we need some darker details, instead of focusing on ''safer'' dark areas. Note that we still need to stop the algorithm for some average value for the new lines, but this time we could be more conservative, and already for some low values on the grayscale spectrum we get some improved result.

<p align="center">
<img src="\assets\img\2022-07-29-string-art\ada_refined.png"  style="width:90%; display: block; margin-left: auto; margin-right: auto;" ><br>
<em>String art portrait of Ada Lovelace with 256 pins, obtained with the refined algorithm, stopping at different values of the grayscale spectrum.</em>
</p>

This is already an improvement from the previous attempt, as we can see some details as the eyes and the hair, but it is still barely recognizable.

### The final algorithm

Note how the previous algorithm sketches some details, like the eyes, but then does not define them properly. The reason seems to be that the intermediate image $$I_k$$ loses information about these details too fast, as they quickly get whitened by the previous lines. The next step is then to be more conservative with this whitening process, and instead of subtracting a black line from $$I_k$$ to obtain $$I_{k+1}$$, we subtract a gray one with some opacity. Opacity works multiplicatively, intuitively this means that initially the lines will brighten up faster than bright ones. On the canvas this has the effect of forcing more lines to overlap on darker spots. This means that, if fix a value $$o \in [0,1]$$ for the opacity, we update the image $$I_k$$ as

$$I_k = I_{k-1} - o \cdot \ell_{p_{k-1}, p_k}, \text{ for } k > 0.$$

This worked like a charm, here are some results for different values of opacity.

<p align="center">
<img src="\assets\img\2022-07-29-string-art\ada_final.png"  style="width:90%; display: block; margin-left: auto; margin-right: auto;" ><br>
<em>String art portrait of Ada Lovelace with 256 pins, obtained with the final algorithm, for different opacity values.</em>
</p>

For the picture at the top of the page I have picked a value of $$0.15$$.

### Is the greedy approach restrictive?

One might argue that this greedy approach might be too rough, what if instead of picking the best line, we pick one which is slightly worse but which allows us to draw better lines later on? This makes absolutely sense, and such an algorithm would surely improve the final result. We could also derive from the previous one. Instead of minimizing the brightness of a line from a given pin over all the possible choices for the following pin, we could at each step minimize the same quantity over all the possible choices of both pins. In this way we could create an image made up of disconnected segments on the canvas, which then we can attempt to reorder to see if they can be concatenated correctly. This is extremely unlikely to be the case, this problem is equivalent to find an [Eulerian path](https://en.wikipedia.org/wiki/Eulerian_path) in the graph defined by all the segments, and a necessary (and sufficient) condition for one to exist is that this graph is connected (extremely likely if there are many pins) and every vertex has even degree (extremely unlikely if there are many pins). But at this point we can force the graph to satisfy this condition by progressively adding lines until all vertices have even degree, which can be done with little impact on the final image. 

The question is how much better the results will get by doing so? We can make an experiment comparing the result of the greedy algorithm with a modified "disconnected" one drawing the best line at each step, without bothering checking that the latter can be realized with a unique connected string. 

<p align="center">
<img src="\assets\img\2022-07-29-string-art\ada_comparison.png"  style="width:70%; display: block; margin-left: auto; margin-right: auto;" ><br>
<em>String art portrait of Ada Lovelace with 256 pins. Comparison between the greedy algorithm (left) and the modified "disconnected" version picking the best line between any two pins at each step (right). </em>
</p>

While differences are expected when the number of pins is small, the results are almost identical when the number grows. Since for photorealistic applications the number of pins needs to be in the hundreds, I decided to stick with the greedy - and way faster - algorithm.

# Final touches
At this point I was quite satisfied with the result. As a final touch I felt that it would be against the spirit of the string artist to stretch short lines between neighboring pins, so I added the options to control how close two consecutive pins can be. For the picture at the top of the page the number I have picked is $$32$$, one eighth of the total number of pins, meaning that two consecutive pins on the string path are at least $$32$$ pins away from each other.

<p align="center">
<img src="\assets\img\2022-07-29-string-art\ada_neighbors.png"  style="width:90%; display: block; margin-left: auto; margin-right: auto;" ><br>
<em>String art portrait of Ada Lovelace with 256 pins, obtained with the final algorithm, for different values of minimum distance between two pins.</em>
</p>

Note that this choice has the effect of discoloring outside a circle, while keeping all the details inside of it. Such circle is the one tangent to all the chords between pairs of closest possible pins. 

# Implementation and choice of parameters

The implementation of the algorithm is [available on my GitHub](https://github.com/gabrieleballetti/string-art). It is written in C++ and I have followed quite closely the algorithm described above, the only difference is that instead of modifying the original image $$I_0$$, I use a "draft" image $$D_k$$ where I progressively add all the opaque lines which then I "subtract" from $$I_0$$. Formally, $$D_0$$ is a white image and

$$
\begin{align*}
D_k &= D_{k-1} + \ell_{k-1, k}, &\text{ for } k > 0,\\
I_k &= I_0 - D_{k-1}, &\text{ for } k > 0.
\end{align*}
$$

I added a bunch of arguments to granularly control all the parameters I have discussed so far

```
string_art input.pgm num_pins opacity threshold skipped_neighbors output_scale output.pgm
```

The opacity and threshold arguments can be used to tweak the final result, see the differences in the table below.

<p align="center">
<img src="\assets\img\2022-07-29-string-art\ada_table.png"  style="width:90%; display: block; margin-left: auto; margin-right: auto;" ><br>
<em>String art portraits of Ada Lovelace with 256 pins, for different values of opacity and threshold.</em>
</p>

For parsing the image and the other argument I readapted the [code from Possibly Wrong](https://github.com/possibly-wrong/string-art). The input and output images are square binary "P5" .pgm (portable graymap) images, as it is easy to read and write.

### Results and comparisons

On the [post on Possibly Wrong](https://possiblywrong.wordpress.com/2022/01/22/string-art/) there is a comparison between their result with those from a paper by [Birsak et al](https://onlinelibrary.wiley.com/doi/10.1111/cgf.13359). The following results of my implementation can be compared with the table at the bottom of the post.

<p align="center">
<img src="\assets\img\2022-07-29-string-art\aec.png"  style="width:80%; display: block; margin-left: auto; margin-right: auto;" ><br>
<em>Some string art portraits with 256 pins.</em>
</p>

Regarding the algorithm used, it seems like I have basically reinvented the wheel, as most of the implementations I have looked up seem to follow a similar logic. Birsak et al. - beside several other improvements and technicalities - instead of opacity use a block-based upscaling where a pixel of the original image corresponds to a block, and when multiple threads pass through that block they update the fraction of its area which is covered, then they transform this fraction into a gray tone which is finally compared against the original pixel. This approach has the advantage to automatically set the correct the ''opacity'' when changing the scale factor, while with my implementation they are independent and the opacity is manually set. 

At the end of the day, I am extremely satisfied with the results and the path leading me to the final version, and comparing the outputs with those on the original paper it seems that - and here I might be very wrong - the conversion from list of pins $$p_0, p_1, \ldots$$ to a computer image makes as big of a difference as the algorithm used, at least for these large values. I use a Bresenham line black-or-white approach, where probably a antialiased line with an alpha channel would have made the result sharper. Maybe I could have tried to output the image as a .svg, and let the conversion do the rest. 

### Some trippy animations

Beside the mandatory animated sequence of lines that generate the final image at the top of the page, we can use animations to visualize how the output changes when we let one parameter vary, and fix all the others. The results are indeed trippy!

<p align="center">
<img src="\assets\img\2022-07-29-string-art\ae_pins.gif"  style="width:40%;" ><br>
<em>
String art portrait of Albert Einstein. At each frame a pin is added and the same algorithm is run, up to 256 pins. 
</em>
</p>

<p align="center">
<img src="\assets\img\2022-07-29-string-art\ae_neigh.gif"  style="width:40%;" ><br>
<em>
String art portrait of Albert Einstein with 256 pins. At each frame we reduce the minimum distance allowed between consecutive pins by one, down to one.
</em>
</p>



