# Monads
:ohno:
## Basic

Basically a way of plumming together some global state.
Basic functions:
``` Haskell

-- Computes a, throws out result value, computes b in resulting *world* from a
(>>) :: M a -> M b -> M b

-- Computes first arg, feeds result and work into the second argument.
(>>=) :: M a -> (a -> M b) -> M b

-- Can define (>>) in terms of (>>=)
(>>) a b = a >>= (\_ -> b)

-- Finally return which just starts the monad off with a value
return :: a -> M a

-- do is just some syntactic sugaring, just simplifies things

eg:
f = getChar >>= ( \ ch1 ->
    getChar >>= ( \ ch2 ->
    return (ch1,ch2) ) )

-- becomes

f5 = do
    ch1 <- getChar
    ch2 <- getChar
    return (ch1,ch2)

-- expansion of do rules

do x
   y
= x >> do y

do a <- x
   y

= do x >>= (\a -> do y)

do x = x
```

Monad class spec:

```Haskell

class Monad m where
    (>>=) :: m a -> (a -> m b) -> m b
    (>>) :: m a -> m b -> m b
    return :: a -> m a
    fail :: String -> m a

```

Can say something is 'Monadic' with:

```Haskell

instance Monad (Maybe a) where
    (Just x) >>= k = k x
    Nothing >>= _ = Nothing
    return = Just
    fail _ = Nothing

```


## Monad Laws

- Left identity : `return a >>= f = f a`
- Right identity : `m >>= return = m`
- Associativity : `(m >>= f) >>= g = m >>= (\x -> f x >>= g)`

## The state monad

Want to make

`newtype State s a = State s -> (a,s)`

We use the record declaration syntax to make:

```Haskell
newtype State s a = State {
    runState :: s -> (a, s)
}
```

Which creates a function

`runState :: State s a -> s -> (a,s)`

Make a monad from this

```Haskell

instance Monad (State s) where
    return a = State (\s -> (a,s))
    m >>= k = State (\s -> let (a,s') = runState m s
                           in runState (k a) s')

-- simple functions
get :: State s s
get = State $ \s -> (s,s)

put :: s -> State s ()
put s = State $ \_ -> ((),s)

```

### Functors

Allows is to apply functions onto its wrapped values

```Haskell
class Functor f where
    fmap :: (a -> b) -> (f a -> f b)
```

Generally `fmap = <$>`

### Applicatives

Functor allows us to apply a function into just one item.
Applicative extends this to any number of arguments. 
Ie fmap `(N arg function) -> <N args> -> out`

Definition:
```Haskell
class Functor f => Applicative f where
    pure :: a -> f a -- have to define how to wrap a default value in a functor
    (<*>) :: f (a -> b) -> f a -> f b
    -- define how to combine a function of N args with the remaining args
```

Example usage:

```Haskell
fmap2 g x y = pure g <*> x <*> y
```



### Monads


## Transformers

