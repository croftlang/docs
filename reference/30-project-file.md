# Project file

croft.toml sits beside the .croft files, like Cargo.toml in Rust. It holds what is about the build, not the domain, so the domain code never has to mention a particular machine.

## What goes in it

For each target, croft.toml says:

- the [adapter](28-adapters.md) that fills each [port](25-ports.md)
- the C files holding the adapters' native functions
- optionally, a `main` service to run instead of the domain, such as a test entry point

Services only ask for ports. They never name adapters. The compiler checks every name in croft.toml against the program.

## An example

The package comes first.

```toml
[package]
name = "dungeon"
version = "0.1.0"
```

Then each target, with its C files and its adapters.

```toml
[target.spectrum_48k]
native = ["platform/spectrum.c"]

[target.spectrum_48k.adapters]
Screen = "SpectrumScreen"
Keyboard = "SpectrumKeyboard"
Files = "DivmmcFiles"
Rooms = "FileStorage"
```

A test target can run a service of its own instead of the domain, with adapters written in Croft.

```toml
[target.test]
main = "RunTests"

[target.test.adapters]
Screen = "NullScreen"
Keyboard = "ScriptedKeyboard"
Rooms = "MemoryStorage"
```

## Without a project file

A project doesn't need one. Without croft.toml, the program runs its domain on one `desktop` target, with checks on and no adapters. It is named after the domain in snake_case, so `domain Logistics` builds `logistics`.

A folder with no domain has nothing to run, but can still be checked.

## Picking a target

When croft.toml has only one target, `croft compile`, `croft run` and `croft size` use it without `--target`.

Given a single file, as in `croft run main.croft`, they use the croft.toml in that file's folder.

## Runtime checks

Every target checks, while the program runs, the [constraints](09-constraints.md) without `or` that the compiler could not work out. When one is broken, the program stops with a message.

`checks = false` turns that off for a target, saving the code and time the checks take on a small machine. A broken rule then goes unnoticed, a number too big for its `Range` is cut down to fit, arithmetic past 32 bits wraps around, and dividing by zero gives 0.

```toml
# The same program, measured and trusted, without its runtime checks.
[target.spectrum_48k_unchecked]
native = ["platform/spectrum.c"]
checks = false

[target.spectrum_48k_unchecked.adapters]
Screen = "SpectrumScreen"
Keyboard = "SpectrumKeyboard"
Files = "DivmmcFiles"
Rooms = "FileStorage"
```

Constraints with `or` are part of what the program does, so they are checked either way.

## Room for bytes and strings

A `Bytes` with no `Length` of its own keeps room for 64 values. `bytes_length` sets how many for a target, from 1 to 65535: fewer on a small machine, more where HTTP bodies are large.

A `Bytes` with a `Length`, and what `Files.read(path, most)` gives back, keep their own room either way.

A string with no `Length` of its own keeps room for 32 characters. That covers a field, a list item, a map key, a `let` that `set` changes, or a line read from the console. `strings_length` sets how many for a target, from 1 to 65535.

```toml
[target.spectrum_48k]
native = ["platform/spectrum.c"]
bytes_length = 32
strings_length = 24
```

A string with a `Length`, shorter or longer, keeps its own room. So one long `Length`, such as the 1024 characters of Std.Http's URL, doesn't make every string as long.

A longer value going into a string with no `Length` stops the program, or is cut short when the target sets `checks = false`.

A string parameter borrows what it is given, so it takes one of any length.

## Native C files

`native` lists the project's own C files for a target. `croft compile` copies them beside the generated C.

`croft run` builds everything with the compiler named by `CC`, or `cc`, as optimised strict C89:

```sh
cc -std=c89 -pedantic -Wall -Wextra -Wdeclaration-after-statement -O2
```

Every warning is an error in the C Croft writes, standard adapters included. In native files the warnings are shown, but don't stop the build.
