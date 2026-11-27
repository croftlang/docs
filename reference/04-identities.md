# Identities

An identity is a distinct type whose only job is to tell entities apart. An `EnemyId` can never be passed where a `RoomId` is expected, however alike the two numbers look.

## Declaring an identity

An identity is declared in the context. After `for`, it names the aggregate or entity it identifies.

```croft
identity RoomId for Room: Int where Range(1, 20)
identity PlayerId for Player: Int where Range(1, 2)
identity EnemyId for Enemy: Int where Range(0, MAX_ENEMIES - 1)
```

## The identity field

The aggregate or entity holds its identity in one field, and that field can have any name.

Other fields of an identity type are references. A `Hall` with `next: RoomId` points at a room, and its own identity is still the field of type `HallId`.

## Entities with the same name

When two aggregates have an entity of the same name, write which one you mean:

```croft
identity EnemyId for Room.Enemy: Int where Range(0, MAX_ENEMIES - 1)
```
