# Algebra

## Vectors

### Basics

For Matrix `M` and Vector `V` of the same order.

`M*V = transpose(V)*transpose(M)`

### Addition
Elementwise addition

### Subtraction
Elementwise subtraction

### Normalising
Keep direction, bring magnitude to 1.

### Dot product
`A.B = Sum of elementwise products = Sigma(ai * bi) for all i`

`U.V = ||U|| * ||V|| * Cos(Theta)`

Where theta is joining angle.

U dot V is drop V onto U.

#### Magnitude in terms of dot
`||A|| = sqrt(A.A)` - neato!

### Cross product
Magic basically


#### Right hand rule
x cross y means x pointing forwards and y left.

Therefore X cross Y pointing up.

`X = [x0,x1,x2]` Except vertically
`Y = [y0,y1,y2]` Except vertically

```
X cross Y = |x1*y2 - x2*y1|
            |x2*y0 - x0*y2|
            |x0*y1 - x1*y0|
```

### Polygins and normals
Determine if a point is in front of or behind a polygon:

`d = n*(P - vi)` for any `vi`

`d > 0` : in front of triangle
`d = 0` : on the triangle plane
`d < 0` : behind the triangle
### Changing basis
