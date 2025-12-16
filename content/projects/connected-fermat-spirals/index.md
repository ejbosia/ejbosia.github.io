---
title: "Connected Fermat Spirals"
date: 2025-12-15
description: "Connected Fermat Spirals"
summary: "An implementation of the connected fermat spirals algorithm in Python."
tags: ["python"]
draft: false
---

# Introduction

I originally worked on this at Boston University. Reading through the code again having worked professionally for a few years, there is a lot I would clean up. Refactoring this code is a good excuse to improve the readability / structure and try using tools like the uv package manager.

{{< github repo="ejbosia/connected-fermat-spirals" showThumbnail=false >}}

{{< figure
    default=true
    src="pikachu.png"
    alt="Pikachu in CFS"
    caption="Everyone's favorite pokemon filled with a connected-fermat-spiral"
    width="250"
    >}}

# Background

The algorithm is defined the [Connected fermat spirals for layered fabrication](https://dl.acm.org/doi/10.1145/2897824.2925958) paper.

The basic idea is filling a shape with spirals often requires multiple, disjoint spirals. By doubling back on the spiral, the input and output to the spiral can be made to be the same location.

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

# Changes from Original Code

I needed to update the Python version, as the original code was targetting Python 3.8 which is EOL. I chose Python 3.12 as a target, but was able to quickly test Python 3.11 using `uv run --python 3.11`.

The most major change was the inclusion of the `Node` class. I was able to use this for both storing the contour and spiral information. Both datatypes require parent-child relationships. This greatly cleaned up the code.

I made a few "software engineering" changes to the code. These were focused on readability.

 - Add docstrings to each file, class, and function.
 - Use type hints throughout the code.
 - Make variable and function names verbose.
 - Use guard clauses instead of nested if statements.
 - Limit the excessive comment use I had originally (*almost every line*) to only when needed.

By removing a lot of the bandaids that were holding the old project together, some of the bugs with spiral generation came back. I plan on solving those in a more stable way. The old project was prone to crashing...

*If you look at the center of Pikachu above, you can see an example of a crossing path.*

# Notes on UV / Windsurf

I like using `uv` a lot. It is very fast. I also like the ability to separate out dev dependencies, such as `matplotlib` for this repo. It also makes trying other versions of python very easy.

I did not get the hang of `Windsurf`... the main issues I had were with tab-autocomplete. It was very slow and often different from what I wanted. The command option was more useful. I ended up returning to PyCharm.
