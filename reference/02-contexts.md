# Contexts

Everything in Croft is declared inside a context. A context is a bounded context: its own small world, where each term has its own meaning.

## Publishing and using terms

A context exports the terms other contexts may import.

```croft
// Rooms, the enemies in them, and moving between them.
context Dungeon {
    export {
        RoomId,
        EnemyId,
        Position,
        Room.EnemyDefeated
    }
}
```

Another context imports only what it needs, with `import { ... } from`. When another name reads better, give the term an alias with `as`.

```croft
context Scoring {
    import {
        EnemyId,
        Room.EnemyDefeated as EnemyDefeated
    } from Dungeon
}
```

Inside `Scoring`, the event is now called `EnemyDefeated`. The alias takes the place of its own name, so `Room.EnemyDefeated` no longer works there.

## Braces or a header

A context can be written in braces, as above. It can also be opened with a header line at the top of a file, which claims the rest of the file.

An `aggregate` header does the same for one aggregate. Here everything below the two header lines belongs to the `Room` aggregate in `Dungeon`:

```croft
// dungeon/room/commands.croft: a header claims the rest of the file
context Dungeon
aggregate Room

command Move(direction: Direction) = {
    ...
}
```

A context, or an aggregate, can be written in several parts. The compiler joins every part before checking them.

## Documenting a context

A comment directly above `context` documents it, like the one above `Dungeon` earlier.

When a context is written in several files, the comment above the part with `export` is the context's doc. The comments above the other parts describe their own file.
