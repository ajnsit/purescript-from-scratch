# Lists

Lists are linked lists. They can be either *empty* or have a single element consed to an existing list.

`cons` is a function which joins a single element with an existing list to create a new list.

Consing a single element to an empty list gives you a singleton list.

## Inbuilt syntax

In Purescript, lists don't have any special syntactic support.
In Haskell, lists have built in syntactic sugar. However apart from that, they are not special in any way.

## Manual definition

We could define a list of integers like so -

```haskell
data List = Empty | Cons Int List
```

Now, an empty list is `Empty`.

A list of a single element (say 1) is `Cons 1 Empty`.

A list of 2 elements is `Cons 1 (Cons 2 Empty)`. Note the order of arguments.

This way we can build a list of any number of elements by simply repeatedly applying `Cons`.


