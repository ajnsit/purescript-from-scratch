# Functions

Functions are the basic unit of abstraction in functional programming. A function has some *inputs* and produces a single *output*.

Since Purescript is *pure*, the function is entirely understood from its inputs and output. There are no *side effects*.

## Expression

An expression is the basic unit of code. It is composed of constants and function calls. An expression can be *evaluated* to produce a single *value*.

## Constants

Constants are literals, some examples are -

Numbers - 2, 4.3 etc.
Booleans - true, false
Arrays - [1,2,3]
Records - {name: "JJ", age: 22}

## Function Application

To apply a function, write the function name, followed by a space separated list of the arguments.
e.g. -

`1 + 2`
`not False`
`sum [1,2,3,4]`

etc.

`+`, `-`, etc. are operators. They are used in an *infix* manner. Functions like `not` and `sum` are used in a *prefix* manner.

It's possible to use a function name in an infix position by surrounding the function name in backticks. This only works with functions with 2 or more arguments.

```haskell
elem 10 [1,5,10,20]
```

Is the same as -

```haskell
10 `elem` [1,5,10,20]
```

Similarly, operators can be used in a prefix position by surrounding them with parenthesis.

```haskell
1 + 2
```

Is the same as -

```haskell
(+) 1 2
```

You can use this fact to define your own operators -

For example -

```haskell
(++) x y = x + y
```

Now you can also use `++` to add numbers -

`12 ++ 34`

## Function application binds strongest

Function application binds stronger than anything else.

So for example -

`sum [1,2,3] + 10`

means that the list will be summed first, and then 10 will be added.


## Creating new bindings

To create a new binding, we just create an equation -

```haskell
one = 1
two = 2
three = one + two
six = three + three
```

Bindings are effectively constants. Bindings once created cannot be changed.

## Function declarations

A binding which is parameterised is called a function.

```haskell
plus x y = x + y
```

This defines a new function which takes two numbers (named `x`, and `y`) and adds them together. The result is returned, none of the input parameters are changed. Remember that Purescript is "pure", and functions cannot have invisible side effects.

## Lambdas

Functions can also be defined without giving them a name. Those are called "anonymous functions", or "lambdas". You define them by using a `\`, which is supposed to approximate the unicode lambda character. We also use `->` arrow instead of `=`.

We could have also defined our `plus` function as an anonymous function -

```haskell
\x y -> x + y
```

Functions are "first class", so we can then assign an anonymous function to a name -

```haskell
plus = \x y -> x + y
```

This is now exactly equivalent to the original definition of `plus`.

Anonymous functions are very useful for functions that you will only use once. For example, to increment all elements of an array you can do -

```haskell
map (\x -> x + 1) [1,2,3,4]
===> [2,3,4,5]
```


