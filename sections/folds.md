# Folds

## Foldr

1. Short circuits
2. Lazy

```haskell
foldr :: Foldable t => (a -> b -> b) -> b -> t a -> b
foldr f z [] = z
foldr f z (x:xs) = f x (foldr f z xs)
```

```haskell
head = foldr const undefined
```

## Foldl

1. Tail call

```haskell
foldl :: Foldable t => (b -> a -> b) -> b -> t a -> b
foldl f z [] = z
foldl f z (x:xs) = foldl f (f z x) xs
```

