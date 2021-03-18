# Data types a la carte

It's a functional pearl by Swierstra, 2008.

```haskell
data Expr = Val Int | Add Expr Expr
```

But now we want to add more constructors. This is not **extensible**.

New approach.

```haskell
newtype Val e = Val Int deriving Functor
newtype Add e = Add e e deriving Functor

type Expr  = Fix (Val :+: Add)
```

Here, `:+:` is defined in GHC.Generics. And it's a product composition of two functors.
This is called `Coproduct` in Purescript, and is defined in `purescript-functors`.

```haskell
data f :+: g a = InL (f a) | InR (g a)
```

or in Purecript -

```haskell
data Coproduct f g a = InL (f a) | InR (g a)
```

`Fix` comes from recursion-schemes, and provides a way to refer to the current type from within the type. i.e. it "ties the knot" on a type level.

```haskell
data Fix f = Fix (f (Fix f))
```

Now we can define a compose operators on `Val` and `Add`.

```haskell
class Eval f where
  eval :: f Int -> Int
```

```haskell
instance Eval Val where
  eval (Val x) = x
```

```haskell
instance Eval Add where
  eval (Add x y) = x + y
```

Note that these definitions are non-recursive (i.e. eval does NOT call eval again). This is because for the purposes of this instance, we are only dealing with `Val Int` or `Add Int`. So the substructure will always be an integer.

However, because we have instances for `Val` and `Add`, these now work with a generic instance for Eval

```haskell
instance (Eval f, Eval g) => Eval (f :+: g) where
  eval (InL x) = eval x
  eval (InR y) = eval y
```

And now, nested constructs will eval correctly.

Let's first define a nested expression that represents `1 + (2 + 3)`.

```haskell
one = InL (Val 1)
two = InL (Val 2)
three = InL (Val 3)
expr = InR (Add one (InR (Add two three)))
```

To make that a bit clearer we can define helper functions -

```haskell
val x = InL (Val x)
add x y = InR (Add x y)
expr = add (val 1 (add (val 2) (val 3)))
```

And now we can evalue it -

```haskell
eval expr
===> 6
```

## Slight improvement -
To make this slightly less cumbersome, we can use `default type signatures` in Haskell.

```haskell
data Expr a = Eval (Val a) | EAdd (Add a)
  deriving (Generic1, Functor, Eval)
```

Here, the instance for `Eval` is derived using `DeriveAnyClass` extension. And we need to give it a default type signature using `DefaultTypeSignatures`.

The actual instance is provided by the generic instance and default implementation below -

```haskell
class Functor f => Eval f where
  eval :: f Int -> Int
  default eval :: (Generic1 f, Eval (Rep1 f)) => f Int
  eval = eval . from1

deriving newtype instance Eval f => Eval (M1 ic f)
deriving newtype instance Eval f => Eval (Rec1 f)
```

Here `Generic1`, `Rep1`, `Rec1`, `M1` are all from `GHC.Generics`.

Now, `InL`, `InR` etc are replaced by a single constructor `Expr`.


