# Hello world

Every language starts here, and Croft isn't going to be the one that breaks tradition. This page gets a program running, so you know your setup works before reading anything else.

## The program

A Croft program starts at its [domain](29-domain.md). This one prints a line through the built-in Console port, and that's all.

```croft
domain Program {
    calls { Console }
} = Console.print("Hello, World")
```

Save it as `hello.croft`. It needs no croft.toml, so you can run it straight away on the `desktop` target:

```sh
croft run hello.croft
```

## Adding a context

Real code lives in [contexts](02-contexts.md). Here the greeting moves into one, which publishes `greet` so the domain can use it.

```croft
context Greeter {
    publish { greet }

    fn greet(): String = "Hello, World"
}

domain Program {
    contexts { Greeter }
    use { greet } from Greeter
    calls { Console }
} = Console.print(greet()) // prints Hello, World
```

The domain lists the contexts it's made of, imports what it needs from them, and names the ports it calls.

## Source files

Source files end in `.croft`. On machines whose file names are limited to 8.3, they can end in `.cro` instead, and a project can mix the two.

The compiler reads either ending in capitals too, as in `CARGO.CRO`.

Without a croft.toml, the program runs on the `desktop` target. Add one to name adapters, add targets or set the program's name. The [project file](30-project-file.md) explains how.

## The croft command

The `croft` command checks, compiles, runs, measures and lays out a project, a folder or a single file. If you haven't got it yet, [install](https://croftlang.com/install) it first.

```sh
croft check .                        # compiler check
croft compile .                      # check and compile to C
croft run .                          # check, compile and run the C
croft size . --target spectrum_48k   # memory each type and function takes
croft fmt .                          # lay out the source files in the Croft style
croft lsp                            # the language server, for editors
```

`croft lsp` is the language server the editor extensions use. It gives you errors as you type, hover, go to definition, completion and formatting.

## A bigger program

The logistics fixture is a larger program to try it on. It is a small cargo shipping system, after the one in Eric Evans' _Domain-Driven Design_.

```sh
croft check tests/fixtures/logistics
croft compile tests/fixtures/logistics --target desktop
```
