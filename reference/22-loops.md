# Loops

Services and adapters can repeat a block. Domain bodies can't: they iterate with list operations such as `map` and `sum` instead, so they always finish.

A domain body that tries gets this error:

```croft
// only services and adapters can loop; use a list method such as `map` here
```

## Looping until break

`loop` runs until `break`. In a service, `set` can change the service's own `let` names, which is how each turn hands the room to the next.

```croft
// Plays turn after turn until the player quits or a turn fails.
service PlayGame(): Result<Unit, GameError> {
    calls { Keyboard, Screen }
} = {
    let room_id = FIRST_ROOM
    loop {
        if Keyboard.quit_pressed() {
            break
        }
        set room_id = PlayTurn(room_id)?
    }
}
```

A loop gives back `Unit`. A `?` inside it leaves the whole service, not only the loop.

## Counted loops

A counted loop runs once for each value its counter's constraints allow, in order. `Range` includes both ends.

```croft
// runs 24 times: rows 0 to 23
loop (let row where Range(0, 23)) {
    Screen.draw_text("", Position { x: 0, y: row * 8 })
}
```

## Growing a list in a loop

A `let` list or map that `set` changes must say with a `Length` how many items it can hold, since the program keeps room for that many.

```croft
// room for 200 items, one for each turn of the loop
let squares: List<Int> where Length(0, 200) = []
loop (let n where Range(1, 200)) {
    set squares = squares.add(n * n)
}
```

Without a `Length`, `croft check` reports an error.

```croft
// doesn't compile: `seen` is changed with `set`, so it needs a `Length`
let seen: List<Int> = []
set seen = seen.add(1)
```

The `let` keeps its constraints whatever `set` gives it. A value known when the program is compiled that breaks one is an error, and so is a counted loop known to add more items than the `Length` allows.

```croft
// doesn't compile: this loop runs 200 times and adds to `few` each time
let few: List<Int> where Length(0, 100) = []
loop (let n where Range(1, 200)) {
    set few = few.add(n)
}
```

A value only known while the program runs stops it with a message naming the service or function, as a parameter's does, such as `Build: let items must satisfy Length(0, 200)`. A target with `checks = false` skips that.
