# Commands

A command is the only way to change an aggregate. A new one is made by a [factory](13-factories.md); after that, every change goes through a command.

## Declaring a command

A command is declared inside its aggregate. It can have a [contract](12-contracts.md), and it has a body.

```croft
// Damages one enemy, and defeats it at zero hit points.
command Attack(enemy_id: EnemyId, damage: Int where Range(1, 20)) {
    enemies.contains(enemy_id) or EnemyMissing { enemy_id }
} = {
    let enemy = enemies.get(enemy_id)
    let remaining = max(0, enemy.hit_points - damage)
    set enemies = enemies.replace(enemy with { hit_points: remaining })
    emit EnemyDamaged { enemy_id, remaining }
    if remaining == 0 {
        emit EnemyDefeated { enemy_id, at: enemy.position }
    }
}
```

## Set and emit

`set` describes the new state of the aggregate's fields.

`emit` raises an [event](11-events.md). You don't list a command's events anywhere: the compiler works out which ones it raises from the body.

## All or nothing

Nothing changes unless the whole command succeeds. If its contract fails, or the result would break one of the aggregate's invariants, the aggregate is left as it was.

To run a command, you [issue it](15-issuing-commands.md).
