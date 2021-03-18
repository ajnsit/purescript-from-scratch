* * *

# Lenses From Scratch

* * *

# The data-lens approach

### Introducing Lens and Store datatypes

A Lens is a pair of 'Getter' and 'Setter' into a data structure. These operations are named `view` and `set`.

```haskell
view :: s -> a
set  :: s -> a -> s
```

Where `s` is the 'whole' and `a` is the 'part'.

Combining them we get -

```haskell
data Lens s a = Lens (s -> a) (s -> a -> s)
```

We can combine the common parameter `s` to get -

```haskell
data Lens s a = Lens (s -> (a, a -> s))
```

The structure `(a, a -> s)` is also known as a `Store` -

```haskell
data Store a s = Store a (a -> s)
```

So then we have -

```haskell
data Lens s a = Lens (s -> Store a s)
```

`Lens` has a `Category` instance. So you can compose them using `Category.(.)`.

**This is the approach used by the `data-lens` library, but *NOT* the `lens` library.**

### Digression 1 - Lens is a "Costate Comonad Coalgebra"

The data-lens approach tells us why Lens is also a "Costate Comonad Coalgebra".

Note: The following section is taken from [tel's lens tutorial](https://www.fpcomplete.com/user/tel/lenses-from-scratch)

A `Comonad w a` is defined as something with operations -

```haskell
extract   :: w a -> a
duplicate :: w a -> w (w a)
```

It turns out that `Store` is a `Comonad` -

```haskell
instance Comonad (Store a) where
  extract (Store a f) = f a
  duplicate (Store a f) = Store a (\b -> Store b f)
```

Now `Store` is also called `Costate Comonad` as it is dual to `State`. Traditionally, a co-something is something with the arrows reversed. However in this case, we reverse `(,)` and `(->)`. So calling Store as Costate is a slight misnomer.

```haskell
State == (a -> (a ,  s))
Store == (a ,  (a -> s))
```

`Store` is also a `Functor` in the same way `(->)` is a Functor -

```haskell
instance Functor (Store a) where
  fmap g (Store a f) = Store a (g . f)
```

A `Coalgebra` for a `Functor` is simply something that constructs the Functor from a 'seed' -

```haskell
type Coalg f a = a -> f a
```

So the Coalgebra for Store is the same as Lens -

```haskell
Coalg (Store a) s == s -> Store a s == Lens s a
```

So effectively Lens is a `Store Coalgebra` or a `Costate Comonad Coalgebra`.

### Digression 2 - Lens Laws

The two lens operations, when adapted to work with the Lens datatype look like this -

```haskell
view :: Lens s a -> s -> a
set :: Lens s a -> s -> a -> s
```

These operations *must* follow the following lens laws -

##### 1. If we get something from `s` and then put it back, it's the same as not doing anything -

```haskell
set l (view l s) s = s
```

In other words - "A lens has no hidden effects".

##### 2. If we put something in, we can get it back out -

```haskell
view l (set l s a) = a
```

##### 3. If we set something, and then 'overwrite' it with something else, it's the same as if I only did the second setting. The first setting has no effect -

```haskell
set l (set l s a) b = set l s b
```

### We need something more

While this definition of a lens works fine (it's the approach used by data-lens package), it has two problems -

1. Ease of use -

    1. To use a Lens, we have to import the Store data structure and associated libraries
    2. We can compose lenses using the Data.Category (.) but to use it we need to hide the one from prelude which only works on plain functions.

2. It is not possible to write a Lens that changes the type of the data structure

These problems are solved in the Lens library using concepts from semantic editor combinators.


# Semantic Editor Combinators

### Basic idea

Note: This section borrows from [Conal's original blog post on Semantic Editors Combinators](http://conal.net/blog/posts/semantic-editor-combinators)

A Semantic Editor Combinator (first introduced by Conal Elliott), is something that 'modifies' something deep within a structure.

The original notion of "modifying" things in functional programming is the `Functor`. It allows us to take a 'container' and replace its 'contents' with something else.

```haskell
fmap :: Functor f => (a -> b) -> f a -> f b
```

Here `f a` is the whole and `a` is the part which we replace with `b`. The resulting structure is `f b`.

However this scheme only works when we have a data type with the shape `f a`, i.e. with the part to be replaced as the last parameter of the whole. Similarly the shape of the return data type is also constrained to `f b`.

We can generalise this pattern using "Semantic Editor Combinators".

```haskell
type SEC s t a b = (a -> b) -> s -> t
```

Where again `s` is the 'whole' and `a` is the 'part'. When `a` is replaced by `b`, the original `s` becomes a `t`. The types here are kept as general as possible to apply to all situations. In specific instances, `s` and `t` usually relate to each other in some way (such as `f a` and `f b`).

### Simple Examples

`first` and `second` from `Control.Arrow` modify the first and the second parts respectively of a pair (when operating on functions).

```haskell
first :: (a -> b) -> (a,x) -> (b,x)
second :: (a -> b) -> (x,a) -> (x,b)
```

Looking at the definition of SEC, these can be represented as -

```haskell
first :: SEC (a,x) (b,x) a b
second :: SEC (x,a) (x,b) a b
```

Similarly we can write a combinator that modifies the output of a function -

```haskell
result :: SEC (a -> b) (a -> c) b c
result = (.)
```

Or a combinator that modifies the argument of the function -

```haskell
argument :: SEC (a -> c) (b -> c) b a
argument = flip (.)
```

Semantic editor combinators don't necessarily act on only one 'point' in a 'structure'.

For example we can write a combinator to modify all elements of a list

```haskell
element :: (a -> b) -> [a] -> [b]
element = fmap
```

### Fmap as an SEC

SEC are strictly more powerful than Functors. This is evident from the fact that `fmap` itself is an SEC. `fmap` is a rather broad SEC that can modify parts of any `Functor` instance.

```haskell
fmap :: Functor f => SEC (f a) (f b) a b
```

For example it can also be used in place of `second` (thanks to the functor instance for tuples) or `result` (thanks to the functor instance for functions).

### The power of the dot - SEC composition

All SEC can be composed using simple function composition. This is `(.)` from the Prelude, and not the one from `Data.Category`.

Multiple `fmap`s can be composed -

```haskell
fmap . fmap . fmap :: (Functor f, Functor g, Functor h) => SEC (f (g (h a))) (f (g (h b))) a b
```

`(.)` itself is an SEC and can be composed -

```haskell
(.) . (.) . (.) :: SEC (c -> d -> e -> a) -> (c -> d -> e -> b) a b
```

In fact different SEC can be freely composed using (.) to modify deeply embedded values in a typesafe manner -

```haskell
deep :: SEC (a, b -> [c]) (a, b -> [d]) c d
deep = second . result . element
```

Where the definition can be read intuitively as - "with the second element of the pair, then with the result of the function, modify all the elements of the list"

This definition seems to have reversed the order of the 'operations'. Intuitively whenever we compose functions together in Haskell, the operation we perform first comes at the end. However with SEC (and with lenses), the order of functions is reversed, and happens to match the record update syntax in procedural languages.

### traverse as an SEC

Another example of an SEC is `Traversable`.

A `Traversable` gives us a function `traverse` which can traverse the structure from left to right. *Any* loop can be represented as an application of `traverse`.

```haskell
traverse :: (Applicative m, Traversable f) => (a -> m b) -> f a -> m (f b)
```

Note: When `m` is a `Monad` and `f` is a List (`[]`) then `traverse` is the same as `mapM`

`traverse` can also be composed with `(.)` and is an SEC -

```haskell
traverse :: (Traversable f, Applicative m) => SEC (f a) (f b) a (m b)
traverse . traverse :: (Traversable f, Traversable g, Applicative m) => SEC (f (g a)) (f (g b)) a (m b)
```

Note that while this is an SEC, unlike the other SEC we saw, the "modification function" passed to the SEC is of the type `a -> m b` instead of the usual `a -> b`.

We can further generalise `traverse` as explained in the below section.

Also note that since `fmap` and `traverse` are both SECs, you can freely combine them. However they start doing strange things so you have to be careful. It's also one of the reasons why the Lens library uses a slightly different formulation from SECs.

# Lens Library

Finally let's look at the actual implementation in the Lens library. It uses something called "Van Laarhoven Lenses". A good way to build intuition for them is by looking at Setters.

## Setters

### A brief recap of SECs

We saw in the previous section how to generalise `fmap` to `SEC`.

Using `fmap`, given a transformation `a -> b`, we can derive `f a -> f b`.

```haskell
fmap :: Functor f => (a -> b) -> f a -> f b
```

However we can get rid of the `Functor` relationship between the input `f a` and the output `f b`, by replacing `f a` and `f b` with arbitrary type variables `s` and `t` respectively. This gave us the type of `SEC` -

```haskell
type SEC s t a b = (a -> b) -> s -> t
```

### From traverse to a Setter

Now let's recall the type of `traverse` -

```haskell
traverse :: (Applicative m, Traversable f) => (a -> m b) -> f a -> m (f b)
```

This is kind of like an SEC (with `s` == `f a` and `t` == `f b`) but with `m` mucking things up. Let's name this new type a `Setter`.

```haskell
type Setter s t a b = Applicative m => (a -> m b) -> s -> m t
```

And the type of traverse becomes -

```haskell
traverse :: Functor f => Setter (f a) (f b) a b
```

These Setter are used to generalise "deep modification" of data structure in the same way as SEC did. Setters are great introduction to Lenses because they have a similar shape and exhibit similar properties.

So how can we use Setters? For that we look at functions that use `traverse` and generalise them for Setters.

### Generalising `fmapDefault` to 'over'

All `Traversable` are also `Functor`, this can be easily seen when `m` is `Identity`. And hence `Traversable` has `Functor` as a superclass. `Data.Traversable` includes a function called `fmapDefault` which uses `traverse` to derive an `fmap`.

```haskell
fmapDefault :: Traversable f => (a -> b) -> f a -> f b
fmapDefault = runIdentity . traverse (Identity . f)
```

*Note:* This allows us to derive a `Functor` instance from a `Traversable` instance using code similar to the following -

```haskell
instance Functor Foo where
  fmap = fmapDefault -- We let the Traversable instance derive fmap

instance Traversable Foo where
  traverse = ...
```

If we pass the Setter used to `fmapDefault` instead of always using `traverse`, we get a function we call `over`.

```haskell
over :: Setter s t a b -> (a -> b) -> s -> t
over :: Setter s t a b -> SEC s t a b
over t f = runIdentity . t (Identity . f)
```

Which means that `over` can take a Setter and generate an SEC out of it. Which can then be used as before.

`over traverse` is basically `fmap` but works only on Traversables. Let's derive a new Setter (we'll call it `mapped`) so that `over mapped` will work on all Functors.

Note that within `over` the type of the Setter is specialised to use `Identity` -

```haskell
type Setter s t a b = (a -> Identity b) -> f a -> Identity (f b)
```

The definition for `mapped` can then be derived by carefully canceling out the effects of `runIdentity` from the definition of `over`.

```haskell
mapped :: Functor f => Setter (f a) (f b) a b
mapped f = Identity . fmap (runIdentity . f)
```

And

```haskell
over mapped f == runIdentity . mapped (Identity . f)
              == runIdentity . Identity . fmap (runIdentity . Identity . f)
              == fmap f
```

Why would you use `over mapped` instead of `fmap`? Because Setters compose just like SEC. So for example, you can modify data embedded within several functors by using `mapped.mapped` or `mapped.mapped.mapped` etc. -

```haskell
mapped :: Setter (f a) (f b) a b
==>
mapped :: (a -> Identity b) -> (f a) -> (Identity (f b))
==>
mapped . mapped :: (a -> Identity b) -> (f (g a)) -> (Identity (f (g b)))
==>
mapped . mapped :: Setter (f (g a)) (f (g b)) a b

over mapped (+1) [1,2,3] ==> [2,3,4]
over (mapped.mapped) (+1) [[1,2],[3]] ==> [[2,3],[4]]
```

Setter can even change the type of the data structure -

```haskell
over (mapped.mapped) length [["hello", "world"], ["iii"]] = [[5,5], [3]]
```

Using composition, Setters can easily be created for any datatype. For example, it's possible to have other setters that aren't functors -

```haskell
both :: Setter (a,a) (b,b) a b
both f (a,b) = (,) <$> f a <$> f b
```

```haskell
first :: Setter (a,c) (b,c) a b
first f (a,b) = (,b) <$> f a
```

And then similarly compose them -

```haskell
over both (+1) (2,3) = (3,4)
over (mapped.both) length (("hello", "world"), ("iii", "iiii")) = ((5,5), (3,4))
```

Another example - `Text` is not a `Functor` because it can not contain values of arbitrary types. However we can easily create a Setter by simply converting it to a String and back -

```haskell
chars :: (Char -> Identity Char) ‐> Text -> Identity Text
chars f = fmap pack . mapped f
```

However note that while using lenses with `Text` gives you a beautiful API, you lose performance. This is because `Data.Text` is very performance conscious and uses *fusion* to achieve this performance. So if you want performant code you should use the (very extensive) API provided by `Data.Text`.

And then we can compose this with other setters for varied functionality -

```haskell
over chars :: (Char ‐> Char) ‐> Text ‐> Text
over (mapped.chars) :: Functor f => (Char ‐> Char) -> f Text ‐> f Text
over (traverse.chars) :: Traversable f => (Char ‐> Char) ‐> f Text ‐>	f Text
```

### Generalising `mapM` to 'mapMOf

`mapM` is defined in `Data.Traversal` in terms of `traverse`. 

```haskell
mapM :: (Traversable f, Monad m) => (a ‐> m b) ‐> f a ‐> m (f b)
mapM f = unwrapMonad . traverse (WrapMonad . f)
```

`WrapMonad` is a simple wrapper over Monads, and is used here because for all `Monad m`, We have `Traversable (WrapMonad m)`.

Now as we did with `fmapDefault`, we can parameterise `mapM` to take a Setter instead. Let's call this `mapMOf` -

```haskell
mapMOf :: Monad m => Setter s t a b -> (a -> m b) -> s -> m t
mapMOf l f = unwrapMonad . l (WrapMonad . f)
```

Of course, just like `over`, we can use `mapMOf` with other Setters.

For example, here's a way of generating all pairs of integers from 1 - 4.

```haskell
mapMOf both (\x -> [x..x+3]) (1,1) ==> [(1,1),(1,2),(1,3), ... ,(4,2),(4,3),(4,4)]
```

### Syntactic "sugar" using `&` in Control.Lens

If we want to 'set' some value directly instead of modifying it (i.e. ignoring the old value altogether), we can use `set`

```haskell
set :: Setter s t a b -> b -> s -> t
```

So

```haskell
set mapped 2 [1,2,3] = [2,2,2]
set (mapped.mapped) 2 [[1,2],[3]] = [[2,2], [2]]
```

Another useful operator is `&` which is just flipped `$`. This works really well with the reversed function order with lenses to chain multiple lens "modifications" on a single data structure. For example -

```haskell
(1,2,3,4) & _2 .- "hello" & _3 .- "world"
===>
(1,"hello", "world",4)
```

Where `.-` is simply an alias for `set`.

There are also a *LOT* of operators and other combinators that are too numerous to cover here.


### Digression 2: Setter Laws

Setter are very similar to `Functor` and must follow similar laws -

##### 1. If we `fmap`/`over` something with `id` it's like we did nothing at all

For functors -

```haskell
fmap id = id
```

For setters -

```haskell
over l id = id
```

##### 2. If we `fmap`/`over` `f` and then `g`, it's like `fmap`/`over` `(f . g)`

For functors -

```haskell
fmap f . fmap g = fmap (f . g)
```

For setters -

```haskell
over l f . over l g = over l (f . g)
```

Note: Unlike functors, the second law of setters doesn't automatically follow from the first.


### 


