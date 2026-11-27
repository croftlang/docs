# Functions

A function is a pure domain service: a calculation that doesn't belong to one value or aggregate. It has a signature, an optional [contract](12-contracts.md), and a body after `=`.

## A simple function

A comment directly above a function documents it. A return type can carry `where` [constraints](09-constraints.md), just like a field.

```croft
// Manhattan distance between two positions.
fn distance(start: Position, end: Position): Int where Range(0, SCREEN_WIDTH + SCREEN_HEIGHT) =
    abs(start.x - end.x) + abs(start.y - end.y)
```

With no return type, a function gives back `Unit`.

## Working on a list

A body can be one expression spread over several lines. This one gives back the identity of the nearest enemy, or `None` when there are no enemies.

```croft
// The enemy closest to a position, or none if there are no enemies.
fn nearest_enemy(start: Position, enemies: List<Room.Enemy>): Option<EnemyId> = enemies
    .min_by(e -> distance(start, e.position))
    .map(e -> e.id)
```

## With a contract

The contract goes in braces between the signature and the `=`. This one says what the caller must never pass in.

```croft
fn average(numbers: List<Int>): Int {
    !numbers.is_empty
} = numbers.sum(n -> n) / numbers.length
```
