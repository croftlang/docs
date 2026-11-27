# Repositories

A repository is a [port](25-ports.md) for keeping one aggregate. You declare its operations, named in the ubiquitous language, and an adapter decides where the aggregates actually live.

## The rules for operations

Every operation works on the aggregate the repository is for. Each one takes or gives back that aggregate, its identity, or plain values, and never another aggregate.

Every operation can fail with StorageError, so swapping one adapter for another never changes a signature. StorageError has three variants:

```croft
// StorageError: StorageFull, StorageDamaged, StorageUnavailable
// StorageDamaged: written by a different version of the program, or damaged:
// a number that doesn't fit its field, a variant or flag that isn't one, a
// map key or set item kept twice, or a broken constraint or invariant
```

StorageError and Storage are built in, and need no import.

## With no body

Written with no body, a repository has a find and a save.

```croft
repository Rooms for Room
// gives: find(id: RoomId): Result<Room, Room.Error or StorageError>
//        save(room: Room): Result<Unit, StorageError>
```

## With its own operations

A repository can declare its own operations instead.

```croft
repository Cargos for Cargo {
    fn find(id: CargoId): Result<Option<Cargo>, StorageError>
    fn awaiting_customs(): Result<List<Cargo>, StorageError>
    fn save(cargo: Cargo): Result<Unit, StorageError>
    fn remove(id: CargoId): Result<Unit, StorageError>
}
```

## Constraints on operations

An operation's parameters can have [constraints](09-constraints.md), as a port's can.

```croft
repository Beds for Bed {
    fn save(bed: Bed where Made): Result<Unit, StorageError>
}
```

One broken while the program runs stops it with a message naming the operation, `Beds.save: bed must satisfy Made`, unless the target sets `checks = false`. When the repository has an adapter of its own, the message names that adapter's operation instead, as a port's does: `ListBeds.save: bed must satisfy Made`.

## Issuing commands through a repository

Issuing a command on `Room(id)` loads and saves through the repository, so the repository needs a find that fails with NotFound, and a save. The no-body repository above has both.

To issue commands on `Cargo(id)`, Cargos would need a find that fails when nothing is kept. Its error includes the aggregate's NotFound and StorageError:

```croft
enum Lookup {
    include Cargo.NotFound
    include StorageError
}
// fn load(id: CargoId): Result<Cargo, Lookup>
```

## Two ways to keep aggregates

An [adapter](28-adapters.md) written for the repository keeps aggregates its own way, such as in table columns.

```croft
// how cargos are kept, written for the repository
adapter PostgresCargos for Cargos = {
    fn find(id) = native pg_find_cargo(id)
    fn awaiting_customs() = native pg_cargos_awaiting_customs()
    fn save(cargo) = native pg_save_cargo(cargo)
    fn remove(id) = native pg_remove_cargo(id)
}
```

Or croft.toml can bind the repository to an adapter for the built-in Storage port. Storage keeps the bytes the compiler makes from each aggregate, under a key such as `rooms12`.

```croft
port Storage {
    fn load(key: String where Length(1, 8)): Result<Option<Bytes>, StorageError>
    fn save(key: String where Length(1, 8), data: Bytes): Result<Unit, StorageError>
    fn remove(key: String where Length(1, 8)): Result<Unit, StorageError>
}
```

The Storage adapter can be MemoryStorage, which is the default, FileStorage, or one of the program's own.

```croft
// where the bytes live
adapter RedisStorage for Storage = {
    fn load(key) = native redis_get(key)
    fn save(key, data) = native redis_set(key, data)
    fn remove(key) = native redis_del(key)
}
```

## What Storage can provide

Storage can only provide operations of three shapes:

- find by identity, giving back an Option or failing with the aggregate's NotFound error
- save the aggregate
- remove by identity

A repository with any other operation needs an adapter of its own, and `croft check` says so for each target that binds it to Storage.

## Reading back

Storage keeps bytes from outside the program, which can be damaged or changed. So finding an aggregate checks everything it reads back:

- each number fits its field and its `Range`
- each enum tag names a variant
- each `Bool` and option is true or false
- no map key or set item is kept twice
- every field constraint and invariant holds

Anything else gives back `StorageDamaged`, on every target, whatever its `checks` setting.

## Choosing in croft.toml

Each target in the [project file](30-project-file.md) picks how each repository is kept. FileStorage saves through the Files port, so a target that uses it also needs an adapter for Files.

```toml
[target.spectrum_48k.adapters]
Files = "DivmmcFiles"
Rooms = "FileStorage"      # rooms12.dat for room 12

[target.server.adapters]
Rooms = "MemoryStorage"
Cargos = "PostgresCargos"  # Storage has no awaiting_customs, so Cargos needs its own adapter
```
