# locks in general

## Implementations
- Bus Arbiter
- Atomic instructions
- Load locked / Store Conditional
- False sharing
- testAndSet lock
- testAndTestAndSet lock
- Ticket lock
- Array Based Queuing lock
- MCS lock
- Reader Writer lock

## Bus Arbiter

Singular bus arbiter which moderates shared memory access.
If a cpu wants access it asserts it's bus req signal with:
`/BREQn` where n is the cpuid.

Grants round robin with `/BRGRNTn` bus cycle by bus cycle granting.

Necessary for Atomic instructions to be useful

## Atomic instructions

Does a `RMW` in one instruction. Must however assert it's `/LOCK` signal.
Normally just writes to the cache thanks to cache coherency yay!

### IA32/x64 atomic instructions

`xchg eax, lock` exchanges lock with register eax asserts the `/LOCK`

A bunch of other ones:
```Asm
bts
btr
btc
xadd
cmpxchg
cmpxchg8b
cmpxchg16b
inc
dec
not
neg
add
adc
sub
sbb
and
or
xor
```
Must be prefixed with lock, eg:
```Asm
lock
xadd reg, mem
```

## Load locked and store conditional
Predecessor to transactional memory.

Each cpu has a *Lock flag* and a *lock physical address register*.

### Example

## Load locked & SC
### Load locked
```Asm
LL Rn, va ; register, virtual address
```

Equivalent to atomically doing:
```C++
lockFlag = 1
lockPhysicalAddressRegister = physicalAddress(va)
Rn = [va]
```

### Store Conditional
```Asm
SC Rn, va ; same as ll
```

Equialent to atomically doing:
```C++
if (lockFlag = 1)
    [va] = Rn
Rn = lockFlag  // used to tell if it succeeded
lockFlag = 0
```

Snoops on bus and watches for writes to the physical address and invalidates the 
lockFlag if a write happens to the address.

## False sharing

Imagine an array that fits inside a cache line.
Each thread accesses individiual sections of the arrays.
But this results in a lot of bus traffic since they each pull the whole array.
Answer: 
```C++
void* ALIGNEDMA::operator new(size_t sz) { // aligned memory allocator
    sz = (sz + lineSz - 1) / lineSz * lineSz; // make sz a multiple of lineSz
    return _aligned_malloc(sz, lineSz); // allocate on a lineSz boundary
}

void ALIGNEDMA::operator delete(void *p) {
_aligned_free(p); // free object
}
```

aligned malloc of each member

## TestandSet lock

Code:
```Asm
wait mov eax, 1
    xchg eax, lock
    test eax, eax
    jne wait

    ; critical stuff

    mov lock, 0a
```
Works because `xchg` is serialising, (RMW instruction).

In C++ we use the intrinsic: `InterlockedExchange(&lock, 1)` to do this


## TestAndTestAndSet lock

Optimistic
```C++
while (InterlockedExchange(&lock, 1))
    while (lock == 1)
        _mm_pause();
```

```C++
do {
    while (lock == 1)
        _mm_pause();
} while (InterlockedExchange(&lock, 1))
```

### Notes
lock is in the cache, therefore reduces a tonne of bus traffic. Only attempts to take lock when it has been freed.

Generates a lot of bus traffic when the lock is invalidated (everyone grabs at it)

### pause

Signal to processor that it's a spin wait loop. A bit like an fence.

### Exponential backoff

Solves the bus traffic issues of TestAndTestAndSet locks

```C++
d = 1; // initialise back off delay
while (InterlockedExchange(&lock, 1)) { // if unsuccessful…
    delay(d); // delay d time units
    d *= 2; // exponential back off
}
```


## Ticket lock

```C++
class TicketLock {
public:
    volatile long ticket; // initialise to 0
    volatile long nowServing; // initialise to 0
};

void acquire(TicketLock *lock) { // acquire lock
    int myTicket = InterlockedExchangeAdd(&lock->ticket, 1); // get ticket [atomic]
    while (myTicket != lock->nowServing) // if not our turn…
        delay(myticket - lock->nowServing); // delay relative to…
} // position in Q

void release(TicketLock *lock) { // release lock
    lock->nowServing++; // give lock to next thread
}
```

Benefit is delay is proportional to position in queue.
Also fair.
Still polling shared location `nowServing`. This generates a lot 
of bus traffic with `write invalidate` cache coherency protocols (MESI).
With `write update` the delay is not actually necessary as 
each processor isn't generating its own reads.

## Array based Queuing Lock

```C++
struct lock { // shared data structure
    volatile long slot[N][cacheLineSz/sizeof(long)]; // each slot stored in a different cache line
    volatile long nextSlot; // initially 0
};

void acquire (struct lock *L, long &mySlot) { // mySlot is a thread local variable
    mySlot = InterlockedExchangeAdd(&L->nextSlot, 1); // get next slot
    mySlot = mySlot % N; // make sure mySlot in range 0..N-1
    if (mySlot == N - 1) // check for wrap around and handle by ...
        InterlockedAdd(&L->nextSlot, -N); // subtracting N from nextSlot
    while (L->slot[mySlot][0] == 0); // wait for turn
}

void release (struct lock *L, long mySlot) {
    L->slot[mySlot][0] = 0; // initialise slot[mySlot] for next time
    L->slot[(mySlot+1) % N][0] = 1; // pass lock to next thread
}
```

Advantage is threads pass lock through neighbouring array slots (only shared
between two threads!)

## MCS Queue

### Node

- waiting = 0/1
- next = pointer to the next node

## Lock
Is a pointer to the head of the node who currently has the lock.

### Acquire

Set `waiting = 1`.
Attach to end using the lock pointer.

```C++
void acquire(QNode **lock) {
    volatile QNode *qn = (QNode*) TlsGetValue(tlsIndex);
    qn->next = NULL;
    volatile QNode *pred = (QNode*) InterlockedExchangePointer((PVOID*) lock, (PVOID) qn);
    if (pred == NULL)
        return; // have lock
    qn->waiting = 1;
    pred->next = qn;
    while (qn->waiting);
}
```

### Release

Pass lock to next thread by setting `qn->next->waiting = 0`

```C++
void release(QNode **lock) {
    volatile QNode *qn = (QNode*) TlsGetValue(tlsIndex);
    volatile QNode *succ;
    if (!(succ = qn->next)) {
        if (InterlockedCompareExchangePointer((PVOID*)lock, NULL, (PVOID) qn) == qn)
            return;
        while ((succ = qn->next) == NULL); // changed from do … while()
    }
    succ->waiting = 0; // simple case
}
```

### Visualisation

While lock is in use heres how the list looks:

```
     \in crit/
w=0 -> w=0 -> w=1 -> w=1 -> w=1
                        lock^
```

## Reader Writer

Works with an int indicating reader count and if theres a writer. We only ever
allow one writer. Can't get a reader lock while there is a writer. Writer can't
acquire while anyone is reading or writing.
