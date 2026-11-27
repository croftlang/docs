# Aggregates and entities

An aggregate is the consistency boundary around its entities and values. Its invariants hold for all of it at once, and it only changes through its own commands.

## The root and its entities

An aggregate's own fields form the root. An entity is declared inside the aggregate that owns it, and lives only there. Outside the aggregate, an entity is known by its [identity](04-identities.md).

```croft
aggregate Room {
    id: RoomId
    player: Player
    enemies: List<Enemy> where Length(0, MAX_ENEMIES), each Alive
    exit: Position

    entity Player {
        id: PlayerId
        position: Position
        lives: Int where Range(0, 9)
    }

    entity Enemy {
        id: EnemyId
        position: Position
        hit_points: Int where Range(0, 100)
    }
}
```

Something with identity that stands on its own is an aggregate with no entities.

A new aggregate is only made by one of its [factories](13-factories.md).

## Invariants and errors

An aggregate declares its own [errors](16-errors.md), which are part of the ubiquitous language.

Each invariant names the error it fails with. If a [command](14-commands.md) would break one, its changes are thrown away and the aggregate is left as it was.

```croft
aggregate Room {
    invariant ExitClear or ExitBlocked
    invariant EnemiesApart or EnemiesOverlap

    error NotFound
    error EnemyMissing
    error ExitBlocked
    error EnemiesOverlap
}
```
