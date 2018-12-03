# Channels

## Implementation

```Haskell

data Item a = Item a (Stream a)
type Stream a = MVar (Item a)

data Chan a = Chan  (MVar (Stream a)) -- read pointer
                    (MVar (Stream a)) -- write pointer


-- create a channel
newChan :: IO (Chan a)
newChan = do
    hole <- newEmptyMVar -- kinda like the head
    readVar <- newMVar hole
    writeVar <- newMVar hole
    return (Chan readVar writeVar)

-- write to a channel
writeChan :: Chan a -> a -> IO ()
writeChan (Chan _ writeVar) val = do
    newhole <- newEmptyMVar
    oldhole <- takeMVar writeVar
    putMVar oldhole (Item val newhole)
    putMVar writeVar newhole

-- *BAD* read channel
readChan :: Chan a -> IO a
    readChan (Chan readVar ) = do
    stream <- takeMVar readVar
    Item val new <- takeMVar stream
    putMVar readVar new
    return val

-- *GOOD* read channel
readChan :: Chan a -> IO a
readChan (Chan readVar _) = do
    stream <- takeMVar readVar
    Item val tail <- readMVar stream
    putMVar readVar tail

-- create a new channel that shares the writer but has a different reader
dupChan :: Chan a -> IO (Chan a)
dupChan (Chan _ writeVar) = do
    hole <- takeMVar writeVar
    putMVar writeVar hole
    newReadVar <- newMVar hole
    return (Chan newReadVar writeVar)

--peeking but it only kinda works apparently
unGetChan :: Chan a -> a -> IO ()
unGetChan (Chan readVar ) val = do
    newReadEnd <- newEmptyMVar
    readEnd <- takeMVar readVar
    putMVar newReadEnd (Item val readEnd)
    putMVar readVar newReadEnd

```
