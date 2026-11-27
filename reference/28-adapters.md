# Adapters

An adapter implements a [port](25-ports.md), in Croft or with native C. Adapters usually live in a Platform context, and croft.toml chooses which targets use each one, so one adapter can serve several.

## Writing one

An adapter's body comes after `=`, like a function's. It provides every operation its port declares, and the types come from the port, so the operations only name their parameters.

```croft
// platform/spectrum.croft
context Platform {
    use { Screen, Keyboard, Key } from Dungeon

    adapter SpectrumScreen for Screen = {
        fn clear() = native zx_cls()
        fn draw_sprite(sprite, at) = native sprite_draw(sprite, at.x, at.y)
        fn draw_text(text, at) = native zx_print_at(at.y / 8, at.x / 8, text)
    }

    adapter SpectrumKeyboard for Keyboard = {
        fn pressed() = native read_key()
    }
}
```

## Native C

Only adapters may call native C functions. A native function gets:

- numbers by value
- each string as a `croft_string_view`, its characters and length, borrowed for the call
- each `Bytes` as a `croft_bytes_view`, its values and length, borrowed the same way
- anything else by pointer
- a pointer for its result, last

So `zx_print_at` above is declared in C like this:

```c
// platform/spectrum.c
void zx_print_at(croft_int row, croft_int column, croft_string_view text, croft_unit *out);
```

A native giving back `Bytes`, alone or in a `Result`, is given a `croft_bytes_writer *` before that last pointer. The writer says where the values go, where their length goes, and how many there is room for, which is as many as the result keeps.

The last pointer is then for the `Result` without the bytes, a `croft_result_unit_error`. For plain `Bytes` there is no last pointer at all. StdioFiles reads a file this way:

```c
void croft_files_read(croft_string_view path, croft_int most, croft_bytes_writer *into, croft_result_unit_error *out);
```

## Adapters with state

Adapters may hold state, changed with `set`. Every adapter field has a starting value, written after its type and constraints.

This one is written in Croft, for tests. It hands out a fixed list of keys, then says Quit.

```croft
adapter ScriptedKeyboard for Keyboard = {
    keys: List<Key> where Length(0, 32) = [Right, Right, Fire]

    fn pressed() = match keys.first {
        Some(next) -> {
            set keys = keys.drop(1)
            Some(next)
        }
        None -> Some(Quit)
    }
}
```
