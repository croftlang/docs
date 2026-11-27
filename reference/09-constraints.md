# Constraints

A constraint is a named, pure check that returns `Bool`. You write a rule once, give it a name from the domain, and attach it wherever the rule applies.

## Declaring a constraint

The first parameter is the value being checked. Any others are the settings you pass when you use it.

```croft
constraint Alive(enemy: Room.Enemy) = enemy.hit_points > 0
```

Constraints can call each other. Pass the value being checked as the first argument.

```croft
constraint OnScreen(position: Position) =
    Range(position.x, 0, SCREEN_WIDTH - 1) && Range(position.y, 0, SCREEN_HEIGHT - 1)
```

Constraints can also be [generic](08-generics.md), as the built-in `Length` below is.

## Range and Length

Two constraints are built in. `Range` limits a number, and `Length` limits a string or a list. Written out, they would look like this:

```croft
constraint Range(number: Int, low: Int, high: Int) = number >= low && number <= high
constraint Length(items: String or List<T>, low: Int, high: Int) = items.length >= low && items.length <= high
```

`Length` with one argument means exactly that long, so `Length(4)` is exactly 4.

## Where constraints go

A constraint attaches to a field, a parameter, a `let`, each item of a list, or a whole value or aggregate as an invariant. You leave out the first argument, because that is the thing it is attached to.

On a field, with `where`:

```croft
hit_points: Int where Range(0, 100)
```

`Range` also sizes a number. This field is kept in one byte, and arithmetic on it is worked out in 16 bits where that is enough.

On a list, and on each of its items with `each`:

```croft
enemies: List<Enemy> where Length(0, MAX_ENEMIES), each Alive
```

`Length` also sizes a string. This field holds 12 characters, not 32:

```croft
title: String where Length(1, 12)
```

So do the strings made from it, such as `title.trim()` or each part of a split. Longer text is cut short, and the program stops and names the field.

On a parameter:

```croft
command Heal(amount: Int where Range(1, 50)) { ... }
```

A parameter's constraint can read the other parameters, and constants, as its arguments:

```croft
constraint AtLeast(number: Int, low: Int) = number >= low

fn span(low: Int, high: Int where AtLeast(low)): Int = high - low
fn pick(low: Int, high: Int, middle: Int where Range(low, high)): Int = middle
```

A field's or a result's constraint can't read parameters like this. Only a parameter's can.

On a `let`, which keeps its constraints whatever `set` changes it to:

```croft
let visited: List<RoomId> where Length(0, MAX_ROOMS) = []
```

A list or map that `set` changes needs a `Length`, which sizes it as a field's does.

## Invariants

A constraint can relate the fields of a whole [value](06-values.md). Inside the value it becomes an invariant, and every value made must keep it.

```croft
constraint Ordered(span: Span) = span.low <= span.high

value Span {
    low: Int
    high: Int

    invariant Ordered
}
```

A constraint can relate the fields of a whole aggregate too. Inside the aggregate it becomes an invariant, with the error it fails with, so a command that breaks it fails.

```croft
constraint ExitClear(room: Room) = room.enemies.all(e -> e.position != room.exit)

aggregate Room {
    invariant ExitClear or ExitBlocked
}
```

## When constraints are checked

The compiler works out every constraint it can. Wherever a value is known when the program is compiled, a broken constraint is an error from `croft check`, and editors show it as you type.

A value is known when it is:

- a literal or a constant
- a list of known values
- arithmetic or `.length` on known values
- a `let` name that nothing in the body changes with `set`
- what a function or translation gives back for known arguments

This holds for arguments to functions, services, commands and factories, for fields in records and `with`, for a value's invariants when all its fields are known, for single values, for enum variants, for the result of a function, and for the values a `let` with constraints starts with or is `set` to.

```croft
constraint Overdose(mg: Int, max: Int) = mg <= max

fn give(mg: Int where Overdose(4000)): Int = mg

// error: `give` is given `5000` for `mg`, which breaks `Overdose(4000)`
let given = give(5000)
```

A call that a function makes in its own body is followed too, when the arguments it was given make that call's arguments known. A constraint broken there is an error at the call that led to it.

```croft
fn inner(n: Int where Range(0, 10)): Int = n
fn outer(n: Int): Int = inner(n + 1)

// error: `outer(10)` calls `inner(11)`, whose `n` breaks `Range(0, 10)`
let reached = outer(10)
```

Values made with their fields, such as `Date { year: 9999, month: 12, day: 31 }`, are followed as well, and so are guards whose condition is known.

A constraint that reads another parameter is worked out with the value given for it.

```croft
// error: `span` is given `2` for `high`, which breaks `AtLeast(low)`, with `low` given `5`
let width = span(5, 2)
```

When either value is only known while the program runs, it is checked then.

Some values are only known while the program runs, such as input or something read from a [port](25-ports.md). These are checked when they reach the constraint. A broken one stops the program with exit code 1, and a message on standard error naming it: `give: mg must satisfy Overdose(4000)`.

This happens on every target, unless its section of [`croft.toml`](30-project-file.md) sets `checks = false`.

## Failing with an error

A constraint on a parameter can end with `or` and an [error](16-errors.md). Breaking it is then not a mistake but a failure the domain expects. The call fails with that error, and the caller [handles it](17-handling-failure.md) like any other.

```croft
error Overdosed

fn give(mg: Int where Overdose(4000) or Overdosed): Result<Int, Overdosed> = mg

// compiles; fails with Overdosed
let given = give(5000)?
```

So `croft check` accepts a call it knows will fail. A constraint with `or` is always checked, whatever the target's `checks` setting.

Only a parameter's constraint can have `or`, whether it is written inline or in the [contract](12-contracts.md). A field, single value, result, loop counter or `let` has nowhere to fail, so its constraints must hold, and so must a value's invariants.

An aggregate's invariant is different. It always names the error its command or factory fails with.
