# Traversable

```haskell
traverse :: Applicative m => Traversable t => (a -> m b) -> t a -> m (t b)
```

```haskell
for :: Applicative m => t a -> (a -> m b) -> m (t b)
forM :: Monad m => t a -> (a -> m b) -> m (t b)

sequence :: Monad m => t (m a) -> m (t a)
sequenceA :: Applicative m => t (m a) -> m (t a)

-- mapAccumL uses foldl
mapAccumL :: (a -> b -> (a,c)) -> a -> t b -> (a, t c)

-- mapAccumR uses foldr
mapAccumR :: (a -> b -> (a,c)) -> a -> t b -> (a, t c)
```

If you have a traversable, you can get an `fmap` or a `foldMap` automatically with -

```haskell
fmapDefault :: Traversable t => (a -> b) -> t a -> t b
foldMapDefault :: Traversable t => Monoid m => (a -> m) -> t a -> m
```

