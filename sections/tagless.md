# Typeclasses and GADTs

## Initial encoding

Consider a data structure that represents nested HTML tags. A straightforward approach might look like the following -

```haskell
data HTML
  = Div [Attrs] [HTML]
  | Button [Attr] [HTML]
  | Text String
  | ...
```

This is called the initial encoding.

## Final encoding

To convert this to final encoding, first convert to GADT notation -

```haskell
data HTML where
  Div :: [Attr] -> [HTML] -> HTML
  Button :: [Attr] -> [HTML] -> HTML
  Text :: String -> HTML
  ...
```

Then convert to a class by parameterising the type of the data structure itself -

```haskell
class HTML html where
  div :: [Attr] -> [html] -> html
  button :: [Attr] -> [html] -> html
  text :: String -> html
  ...
```

## Tagging the "output" type

Let's say you have a class for data structures supporting mathematical operations -

```haskell
class Maths (i :: *) where
  fromInt :: Int -> i
  add :: i -> i -> i
  eq :: i -> i -> i
```

Now, you can write expressions like - `eq (fromInt 3) (add (fromInt 1) (fromInt 2))`. This is equivalent to `3 == 1+2`.

Here, we can see that the expression can represent boolean values as well as integral values.

However, our expression language also allows us to write expressions like the following - `add (eq (fromInt 1) (fromInt 2)) (fromInt 3)`. Which is equivalent to `(1==2) + 3` which doesn't make much sense!

We can use a simple trick however to restrict the set of valid expressions, by "tagging" the type with a "phantom" type parameter representing the output type.

```haskell
class Maths (i :: * -> *) where
  fromInt :: Int -> i Int
  add :: i Int -> i Int -> i Int
  eq :: i Int -> i Int -> i Boolean
```

Now, `add` is restricted to integer expressions, and `eq` is restricted to returning booleans.

## Reifying dictionaries

When you pattern match on GADTs, any dictionaries associated with the data constructor are brought into scope. This is the source of a well known pattern called the "Dict trick" in Haskell which also uses "Constraint kinds" -

To "reify" dictionaries -

```haskell
data Dict (c :: Constraint) where
  Dict :: forall c. c => Dict c
```

Thanks to GADTs, when you pattern match on `Dict`, the constraint is automatically brought into scope.

So you can write a show function that does not require a Show constraint, but instead takes a reified Show dictionary as an argument -

```haskell
showDict :: Dict (Show a) -> a -> String
showDict Dict a = show a
```

See also - type reification using Typeable.

