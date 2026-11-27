# Domain

The domain is the whole program, declared once, outside every [context](02-contexts.md). Like a [function](19-functions.md), it has a [contract](12-contracts.md) and a body.

## The contract

The contract holds the context map: every context, with arrows from upstream to downstream. A context may only use contexts upstream of it, and the arrows may not go round in a circle.

```croft
contexts {
    Dungeon -> Scoring
    Dungeon -> Platform
}
```

The contract also imports what the body needs, and names the [ports](25-ports.md) it calls, with the context they come from.

```croft
use { SetUpRooms, PlayTurn, FIRST_ROOM } from Dungeon
use { SetUpScoring } from Scoring

calls { Keyboard as keyboard } from Dungeon
```

## The body

The body runs first. Any error passed up with `?` stops the program with exit code 1, printing the error's name on standard error.

Here is the whole domain, contract and body together.

```croft
// game.croft
domain Game {
    // upstream -> downstream: a context only uses contexts upstream of it
    contexts {
        Dungeon -> Scoring
        Dungeon -> Platform
    }

    use { SetUpRooms, PlayTurn, FIRST_ROOM } from Dungeon
    use { SetUpScoring } from Scoring

    calls { Keyboard as keyboard } from Dungeon
} = {
    SetUpRooms()?
    SetUpScoring()?
    loop {
        let pressed = keyboard.pressed()
        if pressed == Some(Quit) {
            break
        }
        PlayTurn(FIRST_ROOM, pressed)?
    }
}
```

## Starting data

The body only sees what contexts publish, so each context sets up its own starting data. Dungeon publishes a [service](27-services.md) for the domain to call.

```croft
// in context Dungeon, published for the domain
service SetUpRooms(): Result<Unit, GameError> {
    calls { Rooms }
} = {
    let player = Room.Player { id: 1, position: Position { x: 16, y: 16 }, lives: 3 }
    let room = Room.Open(id: FIRST_ROOM, player: player, exit: Position { x: 232, y: 168 })?
    Rooms.save(room)?
}
```
