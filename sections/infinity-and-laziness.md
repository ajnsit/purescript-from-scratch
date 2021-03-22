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


## Memoisation and sharing

```haskell
fib :: Integer -> Integer
fib 0 = 0
fib 1 = 1
fib n = fib (n-1) + fib (n-2)
```

Now compare the perf characteristics of

```haskell
1. (fib 25, fib 25)
2. let x = fib 25 in (x,x)
3. let x = fib 25 in (1,2)
```

Rules -
1. Let bound variables are shared for the scope of let.
2. Top level definitions are shared for the lifetime of the program, until the compiler knows they are no longer needed, and then they will be GC'd as usual.


Memoisation of fibs -

```haskell
fibs :: [Integer]
fibs = map fib [1..]
```

Now compare the performance of calling -
`take 25 fibs` multiple times. Is it slower only the first time?


