## Types

Every value has a *type*. We have worked with many types already such as `String`, `Int` etc. They are known as **primitive types**.

`String` and `Number` primitive types map directly to Javascript's inbuilt types.

| Type           | Usage
|----------------|-------------------------------
| `String`       | Native Javascript strings
| `Number`       | Native Javascript numbers

Here's a list of all the other primitive types provided by Purescript -

| Type           | Usage
|----------------|-------------------------------
| `Int`          | Integral values, e.g. `1`, `-23`, `0`
| `Boolean`      | Either `true` and `false`
| `Char`         | A single character, e.g. `'c'`, `'@'`


Note that types in Purescript always start with an uppercase letter.

Usually Purescript is able to *infer* the type of expressions automatically, however if you want you can specify the type of the expression with the `::` operator.

```haskell
fib (10 :: Int)
```

Top level entities can have a *type signature* on a separate line, which is the name followed by `::` followed by the type. Example, `foo` has a type signature that specifies its type as `Int`.

```haskell
foo :: Int
foo = 42
```

Apart from primitive types, there are 3 other important inbuilt types - Arrays, Records, and Functions. They will be covered separately.


