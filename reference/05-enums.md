# Enums

An enum is a fixed set of named variants. Use one when a thing can be exactly one of a few known options.

## Simple variants

The simplest enum is just a list of names.

```croft
enum Direction { North, East, South, West }
```

## Variants with data

Variants can carry data, and each one can carry different data, or none.

```croft
enum TreasureKind {
    Coins(count: Int where Range(1, 99))
    Flask
    Key(door: DoorId)
}
```

## Writing a variant

A variant is written with a dot after the enum's name: `Direction.North`, or `TreasureKind.Coins(count: 20)` for one that carries data.
