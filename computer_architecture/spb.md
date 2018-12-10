# Spin, Peterson and Bakery locks

## Spin

Concurrent program verifier

## Peterson

A kind of *fair* spin lock.

Only 2 threads

```C++
int flag[2]; // initially 0
int last;

void acquire(int id) { // id is the thread ID [0 or 1]
    int j = 1 - id; // 0 -> 1 and 1 ->0
    flag[id] = 1; // want lock
    last = id; // other thread has priority
    while (flag[j] && last == id); // NB last == id
}
void release(int id) {
    flag[id] = 0; // release lock
}
```


Properties:
- Deadlock free
- Starvation free
- Fair
- Safe

### Sequential consistency

From leslie lamport:
> A multiprocessor system is sequentially consistent if the result of any
> execution is the same as if the operations of all the processors 
> were executed in some sequential order, and the operations of each 
> individual processor appear in the order specified by its program.

Further:
- Processors memory accesses in order as in program
- accesses made by the different processors are interleaved arbitrarily AND memory accesses are seen in the same order by ALL processors

- Rt -> Wt+n
- Rt -> Rt+n
- Wt -> Rt+n  `Processor ordering`
- Wt -> Wt+n

#### Synchronisation instructions
These resynchronise execution if read aheads are allowed or whatnot.

- LFENCE (Doesn't read ahead past this)
- SFENCE (Flushes writes to L1 cache)
- MFENCE (Combination of both, So flush writes and don't read ahead)

## Bakery

Often called a ticket lock.

Works for N threads

```C++
int number[MAXTHREAD]; // thread IDs 0 to MAXTHREAD-1
int choosing[MAXTHREAD];
void acquire(int pid) { // pid is thread ID
    choosing[pid] = 1;
    int max = 0;
    for (int i = 0; i < MAXTHREAD; i++) { // find maximum ticket
        if (number[i] > max)
            max = number[i];
    }
    number[pid] = max + 1; // our ticket number is maximum ticket found + 1
    choosing[pid] = 0;
    for (int j = 0; j < MAXTHREAD; j++) { // wait until our turn i.e. have lowest ticket
        while (choosing[j]); // wait while thread j choosing
        while ((v = number[j]) && ((v < number[pid]) || ((v == number[pid]) && (j < pid))));
    }
}

void release(int pid) {
    number[pid] = 0; // release lock
}
```



