## Values and Functions

*Values* are things that exist while your program runs. In Purescript, values are immutable and cannot be changed or destroyed once created, but you can create as many new values as you desire.

You can create new values by just assigning them to *names*.

```haskell
one = 1
name = "Joe Smith"
```

Here we have created two new values. One is an integer, and the other a string/text.

*Functions* are the basic unit of abstraction in Functional programming. Think of functions as transformers of data. They accept *input* data and produce *output* data. Note that this is not a destructive operation and you can continue to use the input data even after the function has returned the output.

To create a function, give it a name and provide various cases of input *arguments* it can handle with the corresponding output.

```haskell
not false = true
not true = false
```

## Expressions and Evaluation

*Expressions* are terms which can be *evaluated* to return a single value. Evaluating usually means repeatedly replacing any function invocations with their output.

Expressions that evaluate to the same value are called *equivalent*.

To invoke functions, just separate the name of the function followed by its input arguments, all separated by whitespace. For example this is an *expression* which is evaluates to `true`.

```haskell
not false
```

"*Referential transparency*" is the name of a guarantee provided by Purescript, which says that any expression can be replaced by an equivalent expression, without changing the output or the overall meaning of the program. This allows the programmer to refactor large programs with ease, and also enables the compiler to carry out safe optimising transformations before running the program.

*Example:* the fibonacci function (`fib` for short) when given a number `n`, returns the nth number in the fibonacci number series. Here, the first two numbers in the fibonacci number series are both `1`, and at any position `n` the number is the sum of the previous two numbers. Here's one way to define it in Purescript -

```haskell
fib 0 = 1
fib 1 = 1
fib n = fib (n-1) + fib (n-2)
```

Let's evaluate the following expression by repeatedly replacing sub-expressions which *match* the left hand side of these equations, with the corresponding equivalent expressions on the right hand side of these equations.

```haskell
fib 3
==> fib (3-1) + fib (3-2)
==> fib 2 + fib 1
==> fib (2-1) + fib (2-2) + fib 1
==> fib 1 + fib 0 + fib 1
==> 1 + 1 + 1
==> 3
```

The following expression would evaluate to `89`
```haskell
fib 10
```

Let's evaluate a *chained* function application using our previously derived equivalent expression for `fib 3`.

```haskell
fib (fib (fib 3))
==> fib (fib 3)
==> fib 3
==> 3
```

