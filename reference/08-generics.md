# Generics

Generics are values and functions written once for many types. Write a pair once, and it works for a pair of anything.

## Type parameters

Type parameters are single capital letters, in angle brackets after the name.

```croft
value Pair<A, B> {
    first: A
    second: B
}
```

Functions take them the same way.

```croft
fn swap<A, B>(pair: Pair<A, B>): Pair<B, A> = Pair { first: pair.second, second: pair.first }

fn first_or<T>(items: List<T>, fallback: T): T = items.first.unwrap_or(fallback)
```

## Instances

Each use with different types is an instance, such as `Pair<Int, String>`. The compiler makes and checks each instance as if it were written out by hand, and compiles it to C of its own.

## Working out the types

You rarely write the types out. A call works out its type parameters from what it is given:

```croft
let numbers = Pair { first: 1, second: "one" }   // a Pair<Int, String>
let swapped = swap(numbers)                      // a Pair<String, Int>
```

It also uses the type it must give back. Here the list must hold `RoomId`s, so this `first_or` gives back a `RoomId`:

```croft
let ids: List<RoomId> = [first_or(rooms, FIRST_ROOM)]
```
