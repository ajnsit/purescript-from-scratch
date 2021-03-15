# Free monads

This is an unusually large and rich topic, with several connections to other important topics.
I intent to break it down into multiple overlapping and bite sized parts, and to add connectors between them.

## Advantages of free monads

1. Transform and compose computation
2. Extensible effects
3. Reflection without remorse (sequence of effects). This is what's used in purescript-free.
4. Aspect oriented programming.

```haskell
data Console a = Readline (String -> a) | Putline String
type DSL a = Free Console a
```

## Fix

```haskell
data Fix f = Free (f (Free f))
```

TODO: Consider other fixpoint types -
1. Mu - data, least fixpoint, finite
2. Nu - codata, greatest fixpoint, infinite

## Free

```haskell
data Free f a = Pure a | Free (f (Free f a))
```

## Cofree

```haskell
data Cofree f a = Cofree a (f (Cofree f a))
```

## Loeb's theorem

Consider -

```haskell
fix :: (a -> a) -> a
fix f = let x = f x in x
```

Here the `a -> a` function is a computation that also has access to the final output. i.e. it takes the final output as its input, and returns the final output.

Loeb is a similar structure but with the notion of containers -

```haskell
loeb :: Functor f => f (f a -> a) -> f a
loeb fs = xs where xs = map ($ xs) fs
```

As an example, consider `f a` to be a container that represents a spreadsheet with cells `a`.
Then, `f a -> a` is a function that takes a spreadsheet and computes the value of a single cell.
And, `f (f a -> a)` is a spreadsheet full of computations that have access to the final spreadsheet state.
Essentially the `loeb` function `f (f a -> a) -> f a` then becomes a function which "evaluates" such a spreadsheet to return a spreadsheet full of plain values `f a`.

Let's take another interpretation of Loeb -

Let's consider `f = if isTrue`, and `a = santa exists`
Then the loeb statement becomes -

> If isTrue (if isTrue (santa exists) then (santa exists)) then (santa exists).

Clearly the inner statement is true, hence santa exists! This is known as Loeb's paradox. 


## Codensity

```haskell
newtype Codensity m a = Codensity (forall b. (a -> m b) -> m b)
```

TODO: Explain how this can be used to optimise free monads

Note that this is similar to -

## Continuation monad transformer

```haskell
newtype ContT r m a = ContT { runContT :: (a -> m r) -> m r }
```

TODO: Break out Continuations into a separate topic, however a short explanation follows -

Consider a simple value of type `a`. We can wrap this in a newtype -

```haskell
newtype Identity a = Identity { runIdentity :: a }
```

Converting this to Continuation Passing Style (CPS) we get -

```haskell
newtype IdentityCPS a = IdentityCPS { runIdentityCPS :: forall r. (a -> r) -> r }
```

It's easy to see that the two forms are equivalent (upto isomorphism) by writing conversion functions between them -

```haskell
toCPS :: Identity a -> IdentityCPS a
toCPS (Identity a) = IdentityCPS ($ a)

fromCPS :: IdentityCPS a -> Identity a
fromCPS (IdentityCPS f) = Identity (f (\a -> a))
```

This isomorphism is possible because the result `r` of the continuation `a -> r` is completely unspecified, and hence that severely limits what can be done with it.

Now we consider sums and products -

When `a` is a product `(u,v)`, we have the isomorphism -

```haskell
Identity (u,v)
==> IdentityCPS (forall r. ((u,v) -> r) -> r)
==> IdentityCPS (forall r. (u -> v -> r) -> r)
```

Similarly when `a` is a sum `Either u v`, we have the isomorphism -

```haskell
Identity (Either u v) ==> IdentityCPS (forall r. (Either u v -> r) -> r)
```

Handling the two cases separately we get -

```haskell
Identity (Either u v) ==> IdentityCPS (forall r. (u -> r) -> (v -> r) -> r)
```

So now that we know the general pattern for sums and products we can apply it to any data type, for example -

```haskell
data List a = Nil | Cons a (List a)
```

First expanding the sum -

```haskell
data ListCPS a = ListCPS (forall r. (Nil -> r) -> (Cons a (List a) -> r) -> r)
```

Now noting that `Nil -> a` is the same as `a`, and also expanding the product `Cons a (List a)` we get -

```haskell
data ListCPS a = ListCPS (forall r. r -> (a -> List a -> r) -> r)
```

This is the Scott encoding. To get the church encoding, we also note that `List a` can be folded down to the result `r`. Leading to -

```haskell
data ListCPS a = ListCPS (forall r. r -> (a -> r -> r) -> r)
```

Or switching the order of arguments around -

```haskell
data ListCPS a = ListCPS (forall r. (a -> r -> r) -> r -> r)
```

This CPS'd version of List is the basis for the LogicT transformer where we add a monadic argument `m` into the mix -

## LogicT

```haskell
newtype LogicT m a = LogicT (forall r. (a -> m r -> m r) -> m r -> m r)
```

Why is this called LogicT? Remember that logic programming usually consists of defining facts and some rules for asserting new facts from existing facts, and then program execution is a search through applying the rules to all existing facts until a goal state is reached.

Well, the monadic instance for LogicT allows us to do exactly that.

TODO: Explain further

## Church/Scott encoded free monads

```haskell
```

TODO: Explain

## Freer Monad / Extensible effects

```haskell
data Ffree f a where
  Pure :: a -> FFree f a
  Impure :: f x -> (x -> FFree f a) -> Ffree f a
```

The freer monad needs no `Functor` constraint on `f`.

TODO: Explain

## Recursion schemes

TODO

## Other "free" structures

### Free functor

```haskell
class Functor f where
  map :: (a -> b) -> f a -> f b
```

Converting to GADT syntax (for pedagogical purposes only) -

```haskell
data Functor f b where
  Map :: forall a. (a -> b) -> f a -> Functor f b
```

Renaming `Functor` to `Coyoneda` and moving back to regular ADT syntax we get -

```haskell
data Coyoneda f b = forall a. Coyoneda (a -> b) (f a)
```

### Free Applicative

Similarly we can derive the free applicative -

Ignoring the `Apply` etc. typeclass hierarchy for a bit -

```haskell
class Applicative f where
  pure :: a -> f a
  ap :: f a -> f (a -> b) -> f b
```

Converting to GADT syntax -

```haskell
data Ap f a where
  Pure :: a -> Ap f a
  Ap :: f a -> Ap f (a -> b) -> Ap f b
```

Note that we did not convert it to -

```haskell
data Ap f a where
  Pure :: a -> Ap f a
  Ap :: Ap f a -> f (a -> b) -> Ap f b
```

That's because that formulation will violate (TODO: How? Explain.) the law -

```haskell
ap ff (Pure a) = ap (Pure ($ a)) ff
```

### Free monads are explained at the top


