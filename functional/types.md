# Types

## Inference Systems

> Haskell is polymorphically typed, so types may be universally quantified over sets of type variables.

Build a tree

like:
```
x : t0       z : t1         y : t2      z : t3
        x y : t4                 y z : t5
                x y ( y z) : t6
                \x y z : t7
```

Any subpart shaped like this:

```
x : t0      z : t1
    x y : t2
```

We can infer: `t0 : t1 -> t2`
 
This is the APP rule

Can therefore rewrite it as

```
x : t1->t2    z : t1
        x y : t2
```

Can apply this to the rest of the tree
          
```
x : t1->t4       z : t1         y : t3->t5      z : t3
            x y : t5->t6        y z : t5
                x y ( y z) : t6
                \x y z : t7
```

Another rule:
the two Zs have different type variables but we can assume they'll have the same type.

Furthermore from our knowledge of _Lambda extraction_ means we can aply this rule:

```
       x:A      e:B
            \x.e : A->B
```

Type environment is a map from names to types.
Eg 
```
1 :: Int
[1,2,3] :: [Int]
```

Var rule for adding variables to the mapping
```
Γ ∪ {x : t} means x : t
```

APP rule for adding mappings when doing an application
```
Γe : t' → t     Γe' : t'
        Γe e' : t
```

ABS rule for abstractions from a lambda
```
Γ ∪ {x : t'} means  e : t
Γ means λx.e : t' → t
```

LET rule for let statements
```
Γ meansp e : σ       Γ ∪ {x : σ} meansp e': τ
            Γ meansp let x = e in e' : τ
```

## Advanced types

### Phantom

> A so-called “Phantom type” is a type parameter that isn’t mentioned anywhere in the body of the declaration

```Haskell
newtype Ty a b = T a
```

### Existential

Where we quantify the type in the constructor.

eg:
```Haskell
data HList = HNil
             | forall a . HCons a HList
```

Best to derive a type class or else we can't do anything with `a`s.

```Haskell
data HList = HNil
            | forall a . Show a => HCons a HList
```

Used for implementing private variables in class type objects.

### Generalised Algebraic Data Types

> This is really just a data type where we declare the types of the constructors directly
> Sometimes called 

Explicitly declare the constructors

```Haskell

-- Bad version not sing GADT
data Expr = N Int
            | Add Expr Expr
            | Mult Expr Exper
            | B Bool
            | Eq Expr Expr
            | If Expr Expr Expr
-- ew runtime errors


-- Better caus we can type check it!
data Expr a where
    N :: Int -> Expr Int
    B :: Bool -> Expr Bool
    Add :: Expr Int -> Expr Int -> Expr Int
    Mult :: Expr Int -> Expr Int -> Expr Int
    Eq :: Eq a => Expr a -> Expr a -> Expr Bool
    -- Eq :: Expr Int -> Expr Int -> Expr Bool
    If :: Expr Bool -> Expr a -> Expr a -> Expr a
```

### Kinds

#### Type Kinds


```Haskell
-- Examples

Int :: *
Maybe :: * -> *
```
Basically how many type arguments they need

`*` is built in

#### Data Kinds


```Haskell
data Nat where
    Zero :: Nat
    Succ :: Nat -> Nat

data Vector a (l :: Nat) where
    Nil :: Vector a Zero
    Cons :: a -> Vector a n -> Vector a (Succ n)
```

This means we can encode the length in the type.

But we can't really do operations on this, for example what is the type of append?

##### TypeFamilies

We can write type level functions with type families! (type types?)

```
type family Add (x :: Nat) (y :: Nat) :: Nat
```



### Dependent Types

Type depends on value.
