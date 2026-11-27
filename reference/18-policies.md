# Policies

A policy reacts to an [event](11-events.md) by issuing a [command](14-commands.md), often on another aggregate or in another context. This is how a change spreads without one command touching two aggregates.

## Reacting in the same context

`when` names the event. Inside, `event` is the event that happened, and `event.source` is the identity of the aggregate that raised it.

```croft
policy DropLootOnDefeat when Room.EnemyDefeated {
    issue DropTreasure(at: event.at) on Room(event.source)
}
```

## Reacting in another context

From another context, the event's name starts with the context it comes from. Here the Scoring context gives points whenever an enemy in the Dungeon is defeated.

```croft
// in context Scoring
policy ScoreOnDefeat when Dungeon.Room.EnemyDefeated {
    issue AddPoints(amount: 100) on Scoreboard(MAIN_SCOREBOARD)
}
```

The Dungeon has to [export](02-contexts.md) `Room.EnemyDefeated` for Scoring to see it. Apart from that, the Dungeon doesn't know Scoring exists, which is rather the point.
