# Issuing commands

[Commands](14-commands.md) are issued, never called. You say which command, and `on` says which aggregate it goes to.

## On a value you hold

Issue a command on an aggregate you already hold, and you get the changed aggregate back. Saving it is up to you.

```croft
let attacked = issue Attack(enemy_id: target, damage: 5) on room?
rooms.save(attacked)?
```

## On an identity

Put an [identity](04-identities.md) in brackets after the aggregate's name, and the aggregate is loaded, changed and saved for you in one step.

```croft
const MAIN_SCOREBOARD: ScoreboardId = 1

issue AddPoints(amount: 100) on Scoreboard(MAIN_SCOREBOARD)?
```

## Arguments

Arguments are always named. A command with no parameters drops its brackets.

```croft
issue Reset on Scoreboard(MAIN_SCOREBOARD)?
```

An issued command can fail, so it gives back a `Result`. The `?` passes a failure up, as [Handling failure](17-handling-failure.md) explains.
