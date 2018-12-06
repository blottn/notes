# Mechanics

# Pipeline

## Overview

General pipe:
`Application -> Geometry -> Rasterizer`

## Application
Executes on CPU, full control to developer.

## Geometry
### Fixed function
Vertex stream -> `[transformation & lighting]` -> `[Tri assembly]` -> `[Viewport clipping]`

### Programmable
Vertex stream -> `[Vertex shader]` -> `[Tri assembly]` -> `[Viewport clipping]`

Piped into the rasterizer

### Vertex shader
Performs per vertex operations.
Usually outputs:
- colors
- vectors
- teture coordinates
- vertices

Used to perform model view and projection transformations

- Model
    - Transform *model* coordinates into *world* space
- View
    - Transforms *world* coordinates into *camera* space
- Projection
    - Transforms *camera* space coords into *homogenous* coordinates

#### Vector values
Side note about vectors:

`w = 1` : Position
`w = 0` : Direction


### Clipping 
<img alt="clippy" src="https://vignette.wikia.nocookie.net/joke-battles/images/c/cb/Clippy.png/revision/latest?cb=20151209031540" width="60" height="60"/>

Clipping is the process of culling vertexes not shown by the camera 
because they are outside of the perspective and therefore do not 
need to be rendered.

### Screen mapping

Process of mapping the homogenous coordinates to the screen coordinates 
(eg 0->1024)


## Rasterization Stage

> Rasterization: conversion from 3D vertices in screen-space to pixels on the screen

### Fixed function
Screen space triangles -> `[Rasterizer]` -> `[Texture stages]` -> `[Frame buffer]` -> image

### Programmable
Screen space triangles -> `[Rasterizer]` -> `[Fragment Shader]` -> `[Frame buffer]` -> image

### Rasterizer

`[Triangle setup]` -> `[Triangle Traversal]` -> `[Pixel Shading]` -> `[Merging]`

#### Setup

Vertices converted to triangles.

#### Traversal

Gets pixels for a triangle.
Generates a fragment shader for each visible pixel on a triangle.

#### Shading
Compute colors for each pixel here.

#### Merging
Build up the color buffer 2d array of colors, also depth testing with Z Buffer.

Z buffer is just a 2d array of Z value of closest objects.

#### Double buffer
_aside_ Narrator:
We just swap buffers to display so we dont see rasterisation as it's happening

# Graphics Programming

