---
title: "Hyperbolic Tesselation - Reflection Groups"
date: 2021-05-23T15:34:30-04:00
order: "1"
header:
  teaser: /assets/Portfolio/HyperbolicTess/Teaser.png

categories:
  - Portfolio
tags:
  - Mathematics
  - Topology
  - Graphics
  - Algebra
  - Raytracing
---

This project was written for my masters thesis, as one my first cpp projects; as a result I feel the code has many options for improvement, including accelerating the computation on the GPU. 

## Project summary

It feels a little bit odd to write a blog post on this project seeming that I have already written a whole disertation on the subject, which you can find on [github](https://github.com/LawG4/MastersDissertation/raw/main/MastersDissertaion.pdf). 

The dissertation covers an introduction to reflection groups, and hyperbolic space. Then the concept of fundamental domains is introduced, which is a subset of space bounded between a set of mirrors; the mirrors themselves can also be reflected in order to segment the space. The code portion of this project takes set of mirrors and then tessellates the fundamental domain across the entire plane, the sole purpose of this is to create pretty imagery!

![](/assets/Portfolio/HyperbolicTess/Teaser.png)

## Planned improvements 

Researching the project was certainly fun, but the code certainly is not up to my current standard. Eventually I would like to invest some time into improving it. 

### Hardware acceleration

Currently all calculations are done on one thread on the CPU. Which is a massive bottleneck, each pixel on the screen is associated to some point on the hyperbolic plane. this of course lends itself very easily to being accelerated by a fragment shader.

However, there is still some other considerations. For example the pixels on the edge of the Poincaré disk correspond to infinitely far away on the hyperbolic plane, how do we classify these points? A simple answer would be to place a limit on the search depth for each pixel. Or the program could work the other way, tessellating the plane by iteratively reflecting the triangles once at a time. However hyperbolic triangles are curvy, so ironically a geometry and tessellation stage would be required in the graphics pipeline in order to segment and curve the triangles.

The benefits of using the graphics pipeline would allow for easy texture mapping, which would result in even cooler imagery, although I'm working on my Vulkan renderer first to support this part of the project.

### Image output

The image format selected was PPM, because it was by far the simplest, but also the least useful as it needs to be outputted, Ideally I would like the code to output to the swapchain as it is being generated, so a user has something to watch as the image is drawn. Ontop of that, a png image output with transparency would be the best config.