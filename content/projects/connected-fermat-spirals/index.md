---
title: "Connected Fermat Spirals"
date: 2025-12-16
description: "Connected Fermat Spirals"
summary: "An implementation of the connected fermat spirals algorithm in Python."
tags: ["python"]
draft: false
---

# Introduction

I originally developed this project at Boston University. Reading through the code again having worked professionally for a few years, I found a lot I would clean up. Refactoring this code provided an opportunity to improve the readability and structure while using tools like the uv package manager.

{{< github repo="ejbosia/connected-fermat-spirals" showThumbnail=false >}}

{{< figure
    default=true
    src="pikachu.png"
    alt="Pikachu in CFS"
    caption="Everyone's favorite pokemon filled with a connected-fermat-spiral"
    width="250"
    >}}

# Background

The algorithm is defined in the [Connected fermat spirals for layered fabrication](https://dl.acm.org/doi/10.1145/2897824.2925958) paper.

Filling a shape with spirals often requires multiple, disjoint spirals. By doubling back on the spiral, the input and output to the spiral can be made to be the same location.

{{< figure
    default=true
    src="spirals_unconnected.png"
    alt="Shape filled with Fermat style spirals"
    width="250"
    >}}

This can be inserted into the external spiral to create one continuous path.

{{< figure
    default=true
    src="spirals_connected.png"
    alt="Shape filled with connected Fermat style spirals"
    width="250"
    >}}

The code follows this pipeline to convert fill objects in an image with connected spirals.

{{< mermaid >}}
graph LR;
A[Image]-->B[Polygon];
B-->C[Contours];
C-->D[Spirals];
D-->E[Connected Spirals]
{{< /mermaid >}}

The conversion to **Connected Spirals** requires parent-child relationships between each spiral, because each child spiral can only be connected to its parent. The **Contours** step defines the relationships. These steps all use the `Node` class to store this relationship information.

# Updates from Original Code

I needed to update the Python version, as the original code was targeting Python 3.8 which is end-of-life. I chose Python 3.12, but was able to quickly test Python 3.11 using `uv run --python 3.11`.

One significant change was the inclusion of the `Node` class, which represents a simple tree structure. I was able to use this for both storing the contour and spiral information. Both datatypes require parent-child relationships. This greatly cleaned up the code and allowed more-specific type annotations.

I also made a few "software engineering" changes to the code. These were focused on readability.

 - Added docstrings to each file, class, and function.
 - Used type hints throughout the code.
 - Made variable and function names verbose.
 - Used guard clauses instead of nested if statements.
 - Limited the excessive comment use (*I had commented almost every line*) to only when needed.

# Future Work

There are a few path generation items that could be improved:

1. Remove self intersections in spiral generation and connection (visible in Pikachu example).
1. Improve the uniformity of the space fill.
1. Smooth the sharp corners of the path.

The project could also use more unit test coverage, better logging, and an improved entry script.
