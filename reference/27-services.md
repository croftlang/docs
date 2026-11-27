# Services

Services are the outer layer, where the program meets the world. A service lists the [ports](25-ports.md) it calls, loads aggregates, [issues commands](15-issuing-commands.md) and calls ports.

## A service

This one applies one key press to the room, saves the room and redraws it. It finds the room through the `Rooms` [repository](26-repositories.md), which is a port too, so it goes in `calls` beside `Screen`.

```croft
// Applies one key press to the room, saves it and redraws it.
service PlayTurn(room_id: RoomId, pressed: Option<Key>): Result<Unit, GameError> {
    calls { Screen, Rooms }
} = {
    let room = Rooms.find(room_id)?
    let moved = match pressed {
        Some(key) -> issue Move(direction: direction_for(key)) on room?
        None -> room
    }
    Rooms.save(moved)?
    Screen.draw_sprite(PLAYER_SPRITE, moved.player.position)
}
```

The service never says which adapter fills `Screen` or `Rooms`. That is the [project file](30-project-file.md)'s job.
