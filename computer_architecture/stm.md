# STM

## Transactional memory


Makes tentative changes to l1 cache.
On success changes are made atomically on commit.

### Motivations

- Won't be stalled by other threads stalling.
- Avoids common mutual exclusion issues.
- Out performs best locks.

### New Instructions

- `xbegin` # Start TSX
- `xend`   # End TSX and commit
- `xabort` # Abort changes and end
- `xtest`  # Check if in a TSX

```ASM
xbegin L0
; Transaction instructions

xend

L0:
    ; abort goes here
```
### Abort Conditions

Anyother CPU
- reads a location in its write set
- writes to a location in its read or write set 

There must be a non TSX path if there is an abort.

Abort status will be in `eax`.

### Instrinsics

```C++
int status = _xbegin();
if (status ==  _XBEGIN_STARTED) {
    // do the critical bit
    _xend();
    break;
} else {

}
```

### Nesting

Transactions can be nested. However They are only committed when xend is
executed and there is no nesting.

### Cache stuff
We add an additional T bit to each cacheline to indicate if they are used in a
transaction or not.

Eviction of write set cache lines will cause an abort.
Eviction of read set cache lines only _might_ cause an abort.

### Aborts

Transactions will always abort if you overflow an entire cacheline set.


## HLE

Adds two new instructions:

- `xaquire` must prefix a `XCHG` or a `LOCK` prefix.
- `xrelease` must prefix the lock release instruction.

xaquire and xrelease know if they are in an aborted transaction and either start a
transaction or acquire the lock.
