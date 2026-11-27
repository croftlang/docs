# Ports

A port says what the domain needs from the outside world, such as a screen or a keyboard. It gives the signatures only, never how the work is done, so the domain stays the same whatever machine it runs on.

## Declaring a port

Each operation is a signature with no body. An operation with no return type gives back Unit.

```croft
port Screen {
    fn clear()
    fn draw_sprite(sprite: SpriteId, at: Position)
    fn draw_text(text: String where Length(0, 32), at: Position)
}
```

An operation can give back a value, like any function.

```croft
port Keyboard {
    fn pressed(): Option<Key>
}
```

The work itself is done by an [adapter](28-adapters.md), and the [project file](30-project-file.md) picks which adapter fills each port on each target.

## Contracts on operations

An operation can have a [contract](12-contracts.md), with constraints on its parameters or lines such as `low <= high`. Every caller keeps it, whatever adapter fills the port.

```croft
port Dice {
    fn roll(low: Int, high: Int): Int {
        low <= high
    }
}
```

Broken by known arguments, it is an error from `croft check`:

```croft
// error: `Dice.roll(6, 1)` breaks `low <= high`
let thrown = Dice.roll(6, 1)
```

Otherwise the program stops when the adapter's operation starts, unless the target sets `checks = false`. The message names the adapter's operation, and uses the port's names for the parameters, whatever the adapter calls them: `SeededRandom.roll: low <= high must hold`.

A port has nowhere to fail, so its contract lines can't end with `or` and an error. An operation that can fail gives back a `Result`.

## Calling a port

A [service](27-services.md) lists the ports it calls in `calls`, and calls each one by that name, or by the alias after `as`.

```croft
service Greet() {
    calls { Console as out }
} = out.print("Welcome to the dungeon")
```

## Built-in ports

`Console`, `Arguments`, `Environment` and `Process` are declared in Std.Console, which every program has without an import. Every target supplies them, so croft.toml never names an adapter for them.

They have no adapter, so a broken contract on one of them names the port instead, such as `Process.exit: code must satisfy Range(0, 255)`.

`Arguments` gives the command-line arguments, and is empty on machines without a command line.

```croft
service Start(): Bool {
    calls { Arguments }
} = Arguments.all().contains("--demo")
```

`Console` prints a line.

```croft
service Greet() {
    calls { Console }
} = Console.print("Welcome to the dungeon")
```

It also reads a line, and prints a line on the error output.

```croft
service Echo() {
    calls { Console }
} = match Console.read_line() {
    Some(line) -> Console.print(line)
    None -> Console.print_error("nothing to read")
}
```

`Console.read_line()` gives the next line without its `"\n"` or `"\r\n"`, and None at the end of the input. A target with nothing to read gives None.

A line is never cut short with checks on. One longer than a string stops the program, as an argument longer than a string does. With `checks = false`, as much of the line as a string holds is kept, and the rest of that line is skipped.

`Console.print_error` writes to the error output, kept apart from what `Console.print` writes.

`Environment` gives an environment variable, or None when it isn't set.

```croft
service Home(): String {
    calls { Environment }
} = Environment.get("HOME").unwrap_or("/")
```

`Environment.get(name)` reads the variable through C's `getenv`. A target without environment variables gives None. So does a name no variable can have: an empty one, one longer than 127 characters, or one holding `=` or a NUL byte.

A value is never cut short with checks on. One longer than a string stops the program, as an argument does. With `checks = false`, as much of it as a string holds is kept.

`Process` stops the program at once, with an exit code from 0 to 255.

```croft
service Quit() {
    calls { Process }
} = Process.exit(2)
```

A code known to be outside 0 to 255 is an error from `croft check`. One found only while the program runs stops it with checks on, and keeps its lowest 8 bits with `checks = false`.

A port of the program's own with the same name replaces the built-in one.
