# Cheatsheet

## Pipeline

Opengl pipeline is programmable

```
(vertexes) -> Vertex shader -> Triangle assembly -> clipping -> 
rasterizer -> fragment shader -> buffer -> (images)
```

> mapping takes homogenous coordinates and makes screen coordinates

---

Rasterizer

```
Triangle setup -> triangle traversal -> pixel shading -> merging
```

- setup (make triangles from verts)
- traversal (make pixels out of triangle bits and fragment shade for each)
- pixel shading (compute colors (i guess with fragment shader))
- merging is build up 2d array of colors, also do the z buffer stuff.

## Algebra

### Matrix application
Not commutable but:
`M*V = transp(V) * transp(M)`

### Dot
`A.B = Sum of elementwise products = Sigma(ai * bi) for all i`

*OR*

`U.V = ||U|| * ||V|| * Cos(<joining angle>)`
`Cos` cause if they arent perpendicular then the magnitude becomes more

> A dot B is drop A onto B

### Cross

```
X Cross  Y = |x1*y2 - x2*y1|
             |x2*y0 - x0*y2|
             |x0*y1 - x1*y0|
// ew
```

### Normals

Point behind normal:

`d = n*(P-vi)` for any `vi` in polygon.

### Changing basis

Transform to a basis:
```
|Ux Uy Uz| 
|Vx Vy Vz| * V
|Wx Wy Wz|
```

Transform back is inverse of above which is the transpose because:
The unit vectors are unit length and mutually orthoganol! Magic!

## Viewing

### Axonometric
Axonometric is the most basic parallel projection

#### Isometric
If projection plane intersects the axes at the same length

#### Oblique

Special case of Axonometric where the the z intersection point is the origin. I
think

## Hierarchies i guess

## Illumination

Color depends on:
- geometry of the object.
- position, geometry and color of light sources.
- camera location.
- Surface properties/ materials.
- Scattering by other media

Local illumination - Only the object and light sources taken into account.

Global illumination - Account for all modes of light transport.

View (in)dependent - only sample that which goes throw viewport. Perhaps
simulate entire environment but later choose only that which you want to show.
##### Pros/Cons
- View Dependent
    - Only render that which is visible.
    - Works with all geometry.
    - Can cache some sections.
- View independent
    - Only needs to be done once.
    - Can only calculate view independent parts (diffuse light).

### global algorithms

Easy -> Hard

- zBuffer trickery
    - Compute approx shadows
    - Reflectance from planar surfaces
- Ray Tracing
    - Determine exact shadows
    - Only works with point lights
- Radiosity
    - Approximate solutions
    - no shinyness
    - arbitrary lights
- Path Tracing
    - Using a Monte Carlo solution
    - handles refractance, reflectance
    - arbitrary geometry

### Distribution

Bidirectional Reflectance Distribution Function is used to approximate how light
scatters when it hits a surface. Important for correctly simulating the light.

#### Solid Angle

Steradian. SI unit sr.
Like radians but in 3d.

`Area of the cap / (r^2)`

#### Radiometric Units

Power = Watts
Radiosity = Watts/ metre^2 <- out
Irradiance = Watts/ metre^2 <- in
Radiance = Watts/ metre^2 /sr <- radiosity through a specific part

To get Power from a light at point x:
`Power / (4 pi r^2)` where r is = distance to x.

As the impingement angle increases the amount observed go down:
`E = Cos(theta) * (Phi / 4 pi r^2)` theta is angle from normal.

#### BRDF normally


Wi = incoming direction (Thetat-i, Phi-i)
Wr = reflected direction (Theta-r, Phi-r)

`f(Wi, Wr) = dLr(Wr) / Li(Wi) * Cos(theta) * dWi`

#### Spatially varying Bidirectional Reflectance Distribution Function

Given the observers position `x`:

x is position

Wi = incoming direction (Thetat-i, Phi-i)
Wr = reflected direction (Theta-r, Phi-r)


`fr(x,Wi,Wr)=dLr(x,Wr) / Li(x,Wi)*cos(Thetai)*dWi`

Note this is a general form of the equation. we don't know what Li or Lr is
really.

##### Ideal and rough distributions.

- *Ideal specular* is one beam in means one beam out.
- *Rough specular* is one beam in means one fan out.
- *ideal diffuse*  is one beam in means sphere out.
- *directional diffuse* is one beam in, spehere out + bounce extra in direction it was originally going

#### Actual reflectance equation

`Lr(x,Wi,Wr)=Le(x,Wr) + Integral(fr(x,Wi,Wr)*Li(x,Wi)*cos(Theta-i)*dW-i, with domain Omega)`

`Le` is the emitted light (in the case of light sources this is positive).


### Shading

- `Flat` is once per polygon.
    - looks bad on curved surfaces
    - "Faceted look"
- `Smooth/Gouraud` is once per vert and then interpolate for the rest.
    - Makes surface look curved but edges are still pointy (ouch!).
    - Smoothed with averaging normals.
        - `Nv = ((N1 + N2) / 2) / |(N1+N2) / 2|
    - Big errors with low poly count
- `Phong shading` interpolate normals and calculate for every fragment.
    - Interpolates normals between vertices.
    - Pros/cons:
        - Specular lights
        - Handles Mach bands better
        - Expensive!

#### Mach Banding
Caused by sharp edges in colors. Makes the edges appear even sharper! Appears in
flat shading a lot.


### Illumination
Ambient, lambertian, phong

#### Ambient

Just a constant duh.

#### Lambertian (diffuse in disguise)
Same in all directions so:
`fr(x) = roe-d(x) / pi` - this is our BDRF
`roe-d(x) = Phi-i / Phi-r`

L-r,d(x, ~) = (Row-d(x) / pi) * Cost(Theta) * Phi-source / (4 pi d^2)`

note the irradiance stuff on the right, came from way earlier in notes.


#### Phong (specularity!)

`L-r,s(x,V) = (n+2 / 2pi) * (roe-s) * (cos^n(theta)) * (Ph-s / 4pid^2)`

1. normalization term
2. specular reflectivity
3. cosine lobe
4. Irradiance from lights

#### Attenuation

Sometimes we use an attenuation function instead of the `4 pi r^2` ie `4pi * (a + bd + cd^2)`

#### Blinn Phong

Can be expensive to recalculate reflected angle for every fragment. Instead we
use an approximated reflected vector: `(N.H)^2` Where H is halfway between the
normal and the incidence ray.

#### Different Lights

- Directional
    - Represented by just a direction
- Spot
    - defined by
        - Position
        - Direction
        - Cut off angle
        - Exponent
    - Equation
        - `<Normal irradiance> * ((-I.L) / Cos(Theta))^n 
            - I is incidence angle but towards Light
            - L is light direction
            - n controls sharpness at edges.


## Ray tracing
### Basics
For each pixel trace a ray. And then on impact also trace:
- reflected if it's specular.
- refracted if it's transparent.
- Shadow rays towards lights to determine if it should be rendered.

Equation is now

`L(x,V) = emitted + phong + reflected + refracted`

Forms a binary tree.

Ray defined by:

`r = O + td`

### Intersections

Intersects with a sphere iff 
`f(v) = |v-C|^2 - r^2 = 0`

### Shadows

Send out a sampling of rays instead and check what percent intersect with the
light source.

### Sampling

#### Super Sampling
Instead send out multiple rays for each pixel and average value

#### Adaptive Super Sampling
Subdivide into a grid, send 4 rays to corners, 1 to centre. If values are
similar then done else subdivide and recurse.

Areas with lots of variability are well sampled.

#### Random sampling
Instead randomly sample, take average.

Instead randomly sample in subgrid squares.

#### Bounding Volumes and hierarchies

Use a tree to Represent groups of nodes, reduces number of tests to do.

##### Octrees

Space is a cube, subdivide cube to split objects until there is only leaf nodes.
Done.

Can have long subtrees.
Lots of empty space in tree.

Can set max depth.

## Animation

### Interpolation
(also step isnt mentioned but looks really janky)

#### Linear
Draw straight lines duh
`Li(t) = (1-t)Pi + tPi+1`

#### Spline
Make K-order polynomial for the k+1 points
These may over shoot and cause things to pass through walls etc.

##### curves
- Cubic splines
- Bezier curves
- B splines

### Skinning

Attaching Skin to underlying skeleton
Attach vertices to multiple skeleton parts with different _weights_.

#### Linnear Blend skinning
Just applies weighted average of pulls from each vertex.
Weights are set such that the sum is one.

### Kinematics

Describe the positions of body parts as a function of the joint angles.

#### Forward
Define from the joints up.
Very low level and therefore cumbersome.


#### Inverse
Define from the outer lims in
treated as a blackbox.

Animator specifies end effector positions, computer figures out join angles to
make it happen.

##### Issues
Not always a unique solution.
Not always well behaved see above.
Nonlinear problem.
Might be joint limits.

### Physics based

Set properties + hit go.

### Behavioral

Each boid is given information about the global geometric state.
Example of flocking

#### Seperation

Always drift away from nearby boids.

#### Alignment
Fly in same direction as neightbours.

#### Cohesion
Try to fly closer to the center of the flock as well.

## Mapping

Map a point onto a image sampler to get a color value.
Fakes more depth on objects.

### Corresponder functions 
(Samplers)

- Selects a subset of the image for texturing.
- Says what to do at the boundaries.

### Minification and Maginification

- Nearest Sampling
    - Just choose nearest texel
- Bilinear interpolation
    - Take nearest 4 and average value
    - Creates blurriness :/

Mimaps are already pre scaled versions of the image so it doesn't start to
flicker to badly at distance.
Dynamically chosen based on the depth of the image.

### Bump maps
Illusion of depth through lighting
Generally made using another small grey scale image.
These vaues are used to bump the surface normals around.

### Normal maps
We can just bump the normals around using an array of vectors.

We can also use a texture to map the normals around. Basically use an image
instead of an array of vectors? Nothing special just means it's easier to
visualise the mapping.

### Displacement Mapping

We can apply bumps to the actual vertices either.

- challenges
    - collision detection
    - object intersection
    - foot placement? same as collision detection

### Environment map

So reflective stuff shows the room around it

Think goggles in CSGO and how they show a static image from the map.

Bad on flat surfaces as depends on normals.

