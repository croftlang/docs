# Contracts

A contract says what a piece of code needs and what it promises, and you read it before the body. It is where each domain rule is written once, in the place it applies, with its failures named in the domain's words, which leaves the body free to just do the work.

## Where the contract goes

A contract sits in braces between the signature and `=`. Its [constraints](09-constraints.md) can follow a parameter's type or go in the contract, whichever reads better.

Constraints on a parameter and on the result, inline:

```croft
fn heal(amount: Int where Range(1, 50)): Int where Range(0, 100) = ...
```

The same kind of constraints kept out of a long signature, in the contract:

```croft
fn attack(damage: Int, target: EnemyId): Result<Room, Room.Error> {
    damage where Range(1, 20)
    target where Alive
} = ...
```

A line of the contract such as `from != to` is a constraint without a name.

## A caller's mistake

A constraint without `or` is a mistake by the caller if it is broken. When the values it reads are known, it is an error from `croft check`. Otherwise the program stops while it runs, unless the target sets `checks = false`.

```croft
fn average(numbers: List<Int>): Int {
    !numbers.is_empty
} = numbers.sum(n -> n) / numbers.length
```

Here `average([])` doesn't compile, and a list that is not known until the program runs is checked as it runs.

## An expected failure

A constraint with `or` is a failure the domain expects. It is always checked, and it fails the call with its error, which the caller [handles](17-handling-failure.md).

```croft
command Add(item: ItemId, price: Price, in_stock: Int) {
    in_stock > 0 or OutOfStock { item }
} = {
    set items = items.add(item)
    set total = total with { pence: total.pence + price.pence }
}
```

The caller handles `OutOfStock`. The same works on a parameter, so `give(5000)` compiles, and fails with `Overdosed`:

```croft
fn give(mg: Int where Overdose(4000) or Overdosed): Result<Int, Overdosed> = mg
```

Only a parameter's constraint or a contract line can have `or`. A field or result has nowhere to fail, so its constraints must hold.

## Invariants

An aggregate's invariants are part of its contract. They hold after every [factory](13-factories.md) and [command](14-commands.md). If a command would break one, the command fails and nothing changes.

```croft
aggregate Basket {
    invariant Affordable or Overdrawn
}
```

## Calls

`calls` lists the [ports](25-ports.md) a [service](27-services.md) touches, and it can touch no others. The body calls each port by the name `calls` gives it.

```croft
// called as `Screen.clear()` and `Rooms.find(id)`
service PlayTurn(room_id: RoomId, pressed: Option<Key>): Result<Unit, GameError> {
    calls { Screen, Rooms }
} = ...
```

Like `import`, `calls` can bring a port in from another context and give it an alias with `as`.

```croft
// called as `keys.pressed()`
service ReadKey(): Option<Key> {
    calls { Keyboard as keys } from Dungeon
} = keys.pressed()
```

The difference is that `import` only brings in names, while `calls` also lets the body call them.
