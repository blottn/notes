# parallelism

## primitives

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

Primitive impl would be: `data Chan a = MVar [a]`, not satisfactory as
we want read to block when empty...

[Actual impl](./channels/impl.md)

## STM

Locks are standard way of synchronising programs but are real easy to get wrong.

Transactions are the newest solution (since 2005). Basically the idea is
to optimistically do the operations and only log the memory changes, then
later we decide whether to either all or none of them based on possible
conflicts from the logs.

we introduce the function `atomically :: STM a -> IO a
eg
```Haskell
transferFunds ::  Account -> Account -> Int -> IO ()
transferFunds a1 a2 amount = atomically $ do
    withdraw a1 amount
    deposit a2 amount
```

also a TVar which is just a tentative non blocking var

```Haskell
newTVar :: a -> STM (TVar a)
readTVar :: TVar a -> STM a
writeTVar :: TVar a -> a -> STM ()
```

Then our withdraw becomes:

```Haskell
type Account = TVar Int

withdraw :: Account -> Int -> STM ()
withdraw acc amount = do
    bal <- readTVar acc
    writeTVar acc (bal - amount)
```

This is verifiable by the type system! Can't do this without executing it with
the atomically instruction

We can then right functions like `Retry :: STM ()` which would abort and retry

We can't nest with this though unfortunately.

We do get `orElse :: Stm a -> Stm a -> Stm a` which means try the first
otherwise do the seconda

### Transaction Channels

We might want to communicate with another thread during a STM. Can implement
this with a `TChan` datatype.

```Haskell
data TChan a

newTChan :: STM (TChan a)
writeTChan :: TChan a -> a -> STM ()
readTChan :: TChan a -> STM a
```

Implementations

```Haskell
data TChan a = TChan (TVar (TVarList a))
(TVar (TVarList a))

type TVarList a = TVar (TList a)
data TList a = TNil | TCons a (TVarList a)   
-- explicit TNil for empty this time

newTChan :: STM (TChan a)
newTChan = do
    hole <- newTVar TNil
    read <- newTVar hole
    write <- newTVar hole
    return (TChan read write)

readTChan :: TChan a -> STM a
readTChan (TChan readVar _) = do
    listHead <- readTVar readVar
    head <- readTVar listHead
    case head of
        TNil -> retry -- note blocking implemented with retry
        TCons val tail -> do
            writeTVar readVar tail
            return val

writeTChan :: TChan a -> a -> STM ()
writeTChan (TChan _ writeVar) a = do
    newListEnd <- newTVar TNil
    listEnd <- readTVar writeVar
    writeTVar writeVar newListEnd
    writeTVar listEnd (TCons a newListEnd)

```

## Summary

- STM
    - Composable:
        - atomicity
        - blocking
        - error handling
    - STM is faster on small solutions but generally slower than an MVar solution
- MVar
    - Can guarrantee fairness though
    - Blocked threads can be awoken FIFO
    - Can't be fair using TVar

Each thread maintains a threadlocal log of TVars Reads read from log first,
Writes only write to log. When attempting to commit a log we have to verify it
first, If the old value of a read variable is pointer-equal to the current TVar
value then it passes the validity check. If all pass the the block is added to
memory atomically (no interruptions). If it fails then we restart with a new
log. Retry forces an abort and adds the block to the end of the queue based on
the TVars accessed.
