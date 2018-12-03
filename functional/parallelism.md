# parallelism

## pseq and other functions

System library `Control.parallel` provides the function `par :: a -> b -> b`.
This signals to the compiler that a can be done in parallel (forks off a)

Also `seq :: a -> b -> b` this means strictly evaluate both arguments, generally
a way of forcing strictness in haskell.

Not useful when we don't want to block for the first argument to be executed in
parallel. That's why we have `pseq :: a -> b -> b` which is only strict on b.
Allows a to be evaluated in parallel

Lots of issues with just blindly using par:
- unevaluated/ evaluated computations, laziness 

## Concurrency

`Control.Concurrent` provides:
- `forkIO :: IO () -> IO ThreadId`
- `killThread :: ThreadId -> IO ()`
- `threadDelay :: Int -> IO ()`

These work about as you'd expect

### MVars
These are the basic communication primitive

- Functions
    - `newEmptyMVar :: IO (Mvar a)`
    - `takeMVar :: Mvar a -> IO a`
        - Blocks when it's empty
    - `putMVar :: MVar a -> a -> IO ()`
        - Blocks when full

### Channels
Communication is done via *Chan*s.

- Fifo
- Write however/ whenever
- Read blocks
- Operations
    - `newChan :: IO (Chan a)`
    - `writeChan :: Chan a -> a -> IO ()`
    - `readChan :: Chan a -> IO a`
    - `getChanContents :: Chan a -> IO [a]`
    - `isEmptyChan :: Chan a -> IO Bool`
    - `dupChan :: Chan a -> IO (Chan a)`

Channels can cause deadlocks.
Under the hood they are implemented with `MVars`

Primitive impl would be: `data Chan a = MVar [a]`, not satisfactory as we want read to block when empty... [Actual impl](./channels/impl.md)

## Transactional memory
