# Infinity and Laziness

-- An infinite list of natural numbers

```haskell
[1..]
```

First 10 prime numbers -

```haskell
take 10 (sieve [1..])
```

# Cons

1. Performance is harder to reason about.

2. Thunks and leaking memory (space leak)

Classic example is foldl and foldr. They can build a very large suspended computation that can hang around either forever (i.e. it's never *forced*)  or for a very large time.

3. Debugging is harder.

Printing stuff can force thunks.

