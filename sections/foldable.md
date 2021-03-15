# Foldable

Start with -

```haskell
class Foldable t where
  fold :: Monoid m => t m -> m
```

Then cover -

foldr, foldl (link to folds.md)
foldr1, foldl1
foldMap

Examples -
null, length, elem, maximum, minimum, sum, product etc.

