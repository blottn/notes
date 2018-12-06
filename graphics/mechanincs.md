# Mechanics

## Pipeline

### Overview

General pipe:
`Application -> Geometry -> Rasterizer`

### Application
Executes on CPU, full control to developer.

### Geometry
#### Fixed function
Vertex stream -> `[transformation & lighting]` -> `[Tri assembly]` -> `[Viewport clipping]`

#### Programmable
Vertex stream -> `[Vertex shader]` -> `[Tri assembly]` -> `[Viewport clipping]`

Piped into the rasterizer


#### Vertex shader
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

##### Vector values
Side note about vectors:

`w = 1` : Position
`w = 0` : Direction


#### Clipping 
![clippy](https://vignette.wikia.nocookie.net/joke-battles/images/c/cb/Clippy.png/revision/latest?cb=20151209031540)

### Rasterizer
#### Fixed function
Screen space triangles -> `[Rasterizer]` -> `[Texture stages]` -> `[Frame buffer]` -> image

#### Programmable
Screen space triangles -> `[Rasterizer]` -> `[Fragment Shader]` -> `[Frame buffer]` -> image




## Graphics Programming
