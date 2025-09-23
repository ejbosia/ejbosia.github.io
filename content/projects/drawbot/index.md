---
title: "Drawbot"
date: 2022-01-25
description: "CNC Pen Plotter"
summary: "Experimenting with G-code and pen-plotting."
tags: ["python", "c++", "CAD"]
draft: true
---

## Introduction

I had my old tevo tarantula printer and wanted to learn more about coding. Solution - make a pen plotter!

## Development - Hardware

I did as much as possible to reuse old components. These included the tevo tarantula printer, including motors, linear motion components, and the controller board. I also had some additional precision linear rails components.

I had a rough idea of what I wanted to set up for the pen-plotter. It mostly taking the X-Z frame of the printer and rotating it flat. From previous work on the printer, I already had the belt tensioners set up, so all what I would need to do is design the "tool head".

To do this, I used OnShape because it is free for public use. I designed the brackets to be 3D printed as at this point I had a Prusa MK3S. For moving the pen up and down, I set up a simple cam mechanism, driven by a Nema 17. Is this overkill - yes! Is this lazy - yes! Does it get the job done - yes! I mostly did not waant to come up with a spring mechanism, so gravity was the easiest option to get "consistent" force.

In this configuration, the controls were identical to a normal 3-axis 3D printer. X and Y are controlled by stepper motors. The "Z" in this case is moving the pen up and down. This means the 3D printer board could be reused! The only software inputs needed is G-Code.

## Development - Software

### Path Generation

{{< github repo="ejbosia/drawbot" showThumbnail=false >}}

I started with writing code to take an image and "fill" the image with linear patterns in the same way a 3D printer slices a single layer. I managed to get simple spiral and the "zig-zag" pattern worked out after a bit of trial and error. One fun thing with this project was using [Numba](https://numba.pydata.org/) to speed up the path generation.

### Path Generation - C++

{{< github repo="ejbosia/drawbot-cpp" showThumbnail=false >}}

In an attempt to learn more about C++ and to improve performance, I implemented some of the fill algorithms in C++.

### Drawbot Server

{{< github repo="ejbosia/drawbot-server" showThumbnail=false >}}

As a "challenge" to try using Flask, I set up the worlds simplest file server. This was used to ingest G-code files and run them on the drawbot.

## Results

The drawbot can draw pictures. Nothing too exciting, still cool though!


