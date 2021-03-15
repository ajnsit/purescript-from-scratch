# Morphisms

This section and needs motivating examples, and more details.

## Catamorphism

```haskell
fold :: t a -> b
```

Opposite of Anamorphism.

## Homomorphism

```haskell
map :: t a -> t b
```

## Endomorphism

```haskell
a -> a
```

## Isomorphism

```haskell
a -> b, b -> a
```

## Anamorphism

```haskell
unfold :: a -> t b
```

## Hylomorphism

A combination of anamorphism + catamorphism

## Paramorphism

Reduction like catamorphism, but also gets all the previous elements as args.

## Apomorphism

Opposite of paramorphism.

