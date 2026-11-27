# Handling failure

Issuing a command and finding an aggregate can both fail, so each gives back a `Result`. Croft makes you decide what happens to every failure: pass it up with `?`, or swap it for a more specific error with `or`.

## One error type for the whole body

A body that can fail names its error type, usually an enum that includes the [error sets](16-errors.md) of the aggregates it works with, plus any errors of its own.

```croft
enum GameError {
    include Room.Error
    include Scoreboard.Error
    include StorageError
    KeyboardUnavailable
    ScoreboardUnavailable
}
```

## Passing a failure up with `?`

`?` hands the failure to whoever called this body. It works when the body's error type includes the aggregate's errors, as `GameError` includes `Room.Error` here.

```croft
let room = Rooms.find(room_id)?
issue Attack(enemy_id: target, damage: 5) on Room(room_id)?
```

If `GameError` left out `Room.Error`, both lines would fail with:

```croft
// `GameError` does not include `Room.Error`, so it cannot be passed up here
```

## Failing with a specific error using `or`

`or` replaces the failure with an error you choose. This is handy when the caller doesn't care why the scoreboard went wrong, only that it did.

```croft
issue Reset on Scoreboard(MAIN_SCOREBOARD) or GameError.ScoreboardUnavailable
```

`or` also works on an `Option`, failing when it is `None`, and on a `Bool` check, failing when it is false.

```croft
let item = items.first or Missing
amount <= 100 or TooMany
```

## Every Result is handled

A `Result` that is neither passed up nor the body's result doesn't compile. Forget the `?` and the compiler says:

```croft
// this `Result` is not handled: add `?`, or `or` and an error
```
