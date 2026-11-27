# Errors

An error is a named way that working with an aggregate can fail. Errors are part of the ubiquitous language, so they are named in the domain's words.

## Declaring errors

An aggregate declares its own errors. Each has a name, and can have fields describing the failure. A comment directly above an error documents it.

```croft
aggregate Room {
    error NotFound

    error EnemyMissing {
        enemy_id: EnemyId
    }

    // Room data has two enemies on one square; usually a bad room file.
    error EnemiesOverlap
}
```

The compiler numbers errors automatically, so you never pick the numbers yourself.

## Naming an error with fields

When an error has fields, you fill them in where you name it, for example in a [contract](12-contracts.md):

```croft
enemies.contains(enemy_id) or EnemyMissing { enemy_id }
```

## Messages

Messages shown to the player are not part of the domain. They live in the outer layer, which turns each error into text.

```croft
fn message(error: GameError): String = match error {
    EnemyMissing(enemy_id) -> "That enemy has gone."
    NotFound -> "Room missing."
    _ -> "Something went wrong."
}
```

How to pass errors up, or swap one for another, is in [Handling failure](17-handling-failure.md).
