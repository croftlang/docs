# Body expressions

This page lists everything a body can contain. A body is one expression or a block of lines, and the last line is its result.

## Naming a value

`let` names a value. It never changes.

```croft
let gap = distance(player.position, enemy.position)
```

## Guards instead of else

There is no `else`. An `if` is a guard: when its condition holds, it returns its value and the body ends. When no guard matched, the last line of the body is the result.

```croft
if hit_points == 0 {
    Defeated
}

Fighting
```

A guard whose block only takes steps, such as `set`, `emit` or a port call, runs them and the body carries on.

## Match

`match` chooses by enum variant, and must handle every one.

```croft
match kind {
    Coins(count) -> count * 10
    Flask -> 50
    Key(door) -> 0
}
```

A last arm of `_` handles every variant the arms above it don't.

```croft
match kind {
    Coins(count) -> count
    _ -> 0          // every variant not handled above
}
```

## Lists, sets and maps

Lists are written in brackets, `[a, b]`, or `[]` when empty.

```croft
let corners = [Position { x: 0, y: 0 }, Position { x: 248, y: 184 }]
```

Sets are written the same way, wherever a set is expected. Maps are written as `[key: value]`, or `[:]` when empty. The `Map` and `Set` types come from [Std.Collections](24-standard-contexts.md).

## Lambdas

A lambda is a parameter, `->` and a value: `e -> e.id`. List methods take them to do the iterating, since domain bodies have no [loops](22-loops.md).

```croft
enemies.filter(e -> e.hit_points < 10).map(e -> e.position)

items.sum(item -> points_for(item))
```

A lambda with two or more parameters puts them in brackets.

```croft
items.fold(0, (total, item) -> total + points_for(item))
```

A single parameter never takes brackets. `(e) -> e.id` is an error:

```croft
// a lambda with one parameter is written without brackets: `e -> ...`
```

The full set of list methods is in [built-ins](23-built-ins.md).

## A changed copy

`with` gives a copy of a value with some fields changed.

```croft
player with { lives: player.lives - 1 }
```

## Passing a failure up

`?` passes a failure up, as [handling failure](17-handling-failure.md) explains.

```croft
let room = Rooms.find(room_id)?
```
