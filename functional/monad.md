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

Generally it's aliased to: `fmap = <$>`

### Applicatives

Functor allows us to apply a function into just one item.
Applicative extends this to any number of arguments. 

eg: `fmap (N arg function) -> <N args> -> out`

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

Also there are functions for aplicative such as `*>` that does the same as `<*>`
but just ignores the left value (similarly `<*`)

### Monads

From this we know that Monads are just

```Haskell
class Applicative m => Monad m where
    (>>=) :: m a -> (a -> m b) -> m b
    (>>) :: m a -> m b -> m b
    m >> k = m >>= \_ -> k

    return :: a -> m a
    return = pure

    fail :: String -> m a
    fail s = errorWithoutStackTrace s
```

Also `liftM` has the same type signature as fmap so in _general_ fmap will equal
liftM in your monad instance.

> LiftM lifts a function into a monad

Here's the function `liftM`

```Haskell
liftM :: (Monad f) => (a -> b) -> f a -> f b
liftM f m1 = do { x1 <- m1; return (f x1) }
```

#### State monad impl
```Haskell

instance Functor (State s) where
    fmap = liftM

instance Applicative (State s) where
    pure a = State (\s -> (a,s))

instance Monad (State s) where
    m >>= k = State (\s -> let (a,s') = runState m s
                           in runState (k a) s')
    return = pure
```

## Transformers

### Concurrency monad

This would be implemented like so:

```Haskell
data Thread = Action (IO ()) Thread
    | Fork Thread Thread
    | End

    -- record thingy here
    -- creates continueWith :: CM a -> (a -> Thread) -> Thread
newtype CM a = CM {
    continueWith :: (a -> Thread) -> Thread
}

instance Functor CM where
    fmap = liftM

instance Applicative CM where
    pure a = CM $ \k -> k a

instance Monad CM where
    m >>= f = CM $ \k -> m `continueWith` \x -> f x `continueWith` k
```

```Haskell
print :: Char -> CM ()
print c = CM $ \k -> Action (putChar c) (k ())

fork :: CM a -> CM ()
fork m = CM $ \k -> Fork (thread m) (k ())

end :: CM a
end = CM $ \_ -> End
```

> We can't add more functions easily because monads don't compose

Here's how we could add error functionality

```Haskell
class Monad m => Err m where
    eFail :: m a
    eHandle :: m a -> m a -> m a
```

Great, but we need a way to access the inner functions:

```Haskell
class (Monad m, Monad (t m)) => MonadTransformer t m where
    lift :: m a -> t m a
```

Example impl where we attach error functionality to Maybe:

```Haskell
newtype ErrTM m a = ErrTM (m (Maybe a))

instance Monad m => Functor (ErrTM m) where
    fmap = liftM

instance Monad m => Applicative (ErrTM m) where
    pure a = ErrTM (return (Just a))

instance Monad m => Monad (ErrTM m) where
    (ErrTM m) >>= f = ErrTM $ m >>= r
                        where unwrapErrTM (ErrTM v) = v
                            r (Just x) = unwrapErrTM $ f x
                            r Nothing = return Nothing
```

How to lift an action into the inner monad:

```Haskell
instance Monad m => MonadTransformer ErrTM m where
lift m = ErrTM $ do
                a <- m
                return (Just a)
```

Now we define the error functions:

```Haskell
instance Monad m => Err (ErrTM m) where
    eFail = ErrTM (return Nothing)

    eHandle (ErrTM m1)(ErrTM m2) = ErrTM $ do
        ma <- m1
        case ma of
            Nothing -> m2
            Just _ -> return ma

    runErrTM :: Monad m => ErrTM m a -> m a
    runErrTM (ErrTM etm) = do
        ma <- etm
        case ma of
            Just a -> return a
```

## FRPs and Arrow

Arrows 

> Monads allow us to combine actions in a linear way
> Arrows allow some more interesting combinations


```Haskell
class Arrow a where
    arr :: (b -> c) -> a b c
    (>>>) :: a b c -> a c d -> a b d
```
`Kleisli` takes a monad and wraps it in an arrow

Example Arrow:

```Haskell
-- record: runSF :: SF a b -> [a] -> [b] I think....
newtype SF a b = SF { runSF :: [a] -> [b] }

instance Arrow SF where
    -- arr :: (b -> c) -> a b c
    arr f = SF (map f)

    -- (>>>) :: a b c -> a c d -> a b d
    SF f >>> SF g = SF (f >>> g)
```


Splash in some category theory, Arrows come from category theory.

```Haskell
class Category cat where
    id :: cat a a
    (.) :: cat b c -> cat a b -> cat a c
    (>>>) :: Category cat => cat a b -> cat b c -> cat a c
    (<<<) :: Category cat => cat b c -> cat a b -> cat a c
```

```Haskell
class Category a => Arrow a where
    arr :: (b -> c) -> a b c
    first :: a b c -> a (b, d) (c, d)
    second :: a b c -> a (d, b) (d, c)
```

```Haskell
    -- running two arrows on a pair of values (one arrow on the first, the
other on the second)
    (***) :: a b c -> a b' c' -> a (b, b') (c, c')

    -- running two arrows on the same value
    (&&&) :: a b c -> a b c' -> a b (c, c')
```
