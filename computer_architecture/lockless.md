# Lockless

## Methods

- Obstruction Free
    - guaranteed to complete in a finite number of steps if no other thread executes
- Lock Free
    - iff some thread is guarranteed to make progress in some finite number of steps.
- Wait Free
    - it itself is guarranteed to complete in some not necessarily know no. steps.

Lock based solutions aren't obstruction free (if they sleep no other thread executes)

CAS or LL/SC are normally Lock Free.

Solutions where a blocked thread goes and unblocks itself is wait free as it
doesn't stall.



