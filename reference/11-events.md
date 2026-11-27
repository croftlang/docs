# Events

An event is something that happened, named in the past tense. Other parts of the program, such as [policies](18-policies.md), can react to it.

## Declaring events

An event is declared inside the aggregate that raises it, with fields for what is worth knowing about it.

```croft
aggregate Room {
    event PlayerMoved { to: Position }
    event EnemyDamaged { enemy_id: EnemyId  remaining: Int }
    event EnemyDefeated { enemy_id: EnemyId  at: Position }
}
```

[Commands](14-commands.md) and [factories](13-factories.md) raise events with `emit`.

Every event also carries its aggregate's identity as `event.source`, so the event needs no field of its own for it.

From outside the aggregate, the event's name includes the aggregate's: `Room.EnemyDefeated`.

## When an event counts

An event only counts once its aggregate is saved. If the changed aggregate is never saved, the event is dropped, so policies never react to a change that was not kept.
