# Constants

A constant is a named value fixed at compile time. Give a number or a string a name once, and every use of it says what it means.

## Declaring constants

Constant names are in UPPER_SNAKE_CASE.

```croft
const SCREEN_WIDTH: Int = 256
const SCREEN_HEIGHT: Int = 192
const MAX_ENEMIES: Int = 16
const GAME_TITLE: String = "Dungeon"
```

## Using constants

A constant can be used anywhere an expression can. That includes the arguments of a [constraint](09-constraints.md):

```croft
value Position {
    x: Int where Range(0, SCREEN_WIDTH - 1)
    y: Int where Range(0, SCREEN_HEIGHT - 1)
}
```
