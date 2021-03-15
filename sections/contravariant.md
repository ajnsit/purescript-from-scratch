# Contravariant functors

```haskell
class Contravariant f where
  contramap :: (a -> b) -> f b -> f a
```

Example -

## Composable Loggers

```haskell
newtype Logger f a = Logger (a -> f unit)

instance Contravariant (Logger f) where
  contramap f (Logger a) = Logger (a . f)

instance Semigroup (Logger f a) where
  Logger a <> Logger b = Logger $ \r -> a r *> b r
```


