# Factories

A factory is the only way to make a new aggregate. Because every new aggregate comes from one, it is the place to say what a valid starting state looks like.

## Declaring a factory

A factory is declared inside its aggregate. It takes named arguments, can have a [contract](12-contracts.md), and has a body that gives back the new aggregate. It can raise [events](11-events.md) with `emit`.

```croft
aggregate Room {
    id: RoomId
    player: Player
    enemies: List<Enemy> where Length(0, MAX_ENEMIES), each Alive
    exit: Position

    invariant ExitClear or ExitBlocked

    error ExitBlocked
    event Opened { exit: Position }

    // A new room starts with no enemies; its exit must be clear.
    factory Open(id: RoomId, player: Player, exit: Position) = {
        emit Opened { exit }
        Room { id, player, enemies: [], exit }
    }
}
```

The aggregate is held to its invariants before it exists. If one breaks, the factory fails with that invariant's error, and its events are dropped.

## Calling a factory

Services, policies and functions call a factory by the aggregate's name and its own. They get back a `Result` with the aggregate's errors.

```croft
// in a service
let room = Room.Open(id: FIRST_ROOM, player: player, exit: Position { x: 232, y: 168 })?
rooms.save(room)?
```

Only the context that owns the aggregate calls its factories.

## Writing an aggregate out

An aggregate written out with its fields is only allowed in two places: inside its own factories, and in an [adapter](28-adapters.md) written for its [repository](26-repositories.md), which builds it from what it kept.

Anywhere else, it is an error.

```croft
let room = Room { ... }        // a new `Room` is made by its factory: `Room.Open`
rooms.save(room with { ... })  // a `Room` changes by issuing one of its commands, not with `with`
```

To change a room you already have, issue one of its [commands](14-commands.md).
