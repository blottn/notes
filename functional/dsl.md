# DSLs

## Shallow

- Represent DSL programs as values in the host language (like, say, functions)

- Provide a fixed semantics
- A program in the DSL might consist of calls to library functions

## Deep

- Represent DSL programs as values in the host language (still), but kept abstract
- Use higher-order functions (combinators) to piece together programs
- A program in the DSL might consist of construction of a value to describe the program that is fed to an interpreter
