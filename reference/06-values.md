# Values

Values are value objects. A value is defined only by its fields, two values are equal when their fields are equal, and a value never changes.

## Declaring a value

Values can be declared at context level and shared.

```croft
value Position {
    x: Int where Range(0, SCREEN_WIDTH - 1)
    y: Int where Range(0, SCREEN_HEIGHT - 1)
}
```

A value's fields can be other values and enums.

```croft
value Treasure {
    kind: TreasureKind
    at: Position
}
```

## Making one

Make a value with its type name and its fields.

```croft
let start = Position { x: 10, y: 20 }        // make a new Position
```

## Changing one

A value never changes, so to change one you copy it with `with`, giving the fields that are different. The original stays as it was.

```croft
let moved = start with { x: start.x + 1 }    // copy it, changing x
```

## Invariants

A value can have invariants. These are [constraints](09-constraints.md) on the value as a whole, relating its fields.

```croft
constraint Ordered(span: Span) = span.low <= span.high

value Span {
    low: Int
    high: Int

    invariant Ordered
}
```

Every value made, by its type name or with `with`, must keep them. Making a value can't fail, so an invariant on a value has no `or`.

When the compiler knows one is broken, it's an error from `croft check`.

```croft
let wide = Span { low: 1, high: 9 }
// error: `Span { low: 9, high: 1 }` breaks `Span`'s invariant `Ordered`
let backwards = Span { low: 9, high: 1 }
```

Otherwise, with checks on, a broken one stops the program with a message naming it: `Span must satisfy Ordered`.

Storage checks them on each value it reads back.
