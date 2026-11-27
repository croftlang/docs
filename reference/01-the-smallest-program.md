# The smallest program

The smallest Croft program is one context with one function, and a domain that calls it. It's a good place to check your setup works before reading anything else.

## The context

A [context](02-contexts.md) holds the code. This one has a single function, `add`, and publishes it so the domain can use it.

```croft
context Hello {
    publish { add }

    fn add(a: Int, b: Int): Int = a + b
}
```

## The domain

The [domain](29-domain.md) is where the program starts. It lists its contexts, imports `add`, and prints the answer through the built-in Console port.

```croft
domain Program {
    contexts { Hello }
    use { add } from Hello
    calls { Console }
} = Console.print(add(2, 3).to_string())   // prints 5
```

## Running it

Put both parts in one file, `hello.croft`. It needs no croft.toml, so you can run it straight away on the `desktop` target:

```sh
croft run hello.croft
```

## Source files

Source files end in `.croft`. On machines whose file names are limited to 8.3, they can end in `.cro` instead, and a project can mix the two.

The compiler reads either ending in capitals too, as in `CARGO.CRO`.

Without a croft.toml, the program runs on the `desktop` target. Add one to name adapters, add targets or set the program's name. The [project file](30-project-file.md) explains how.

## The croft command

The `croft` command checks, compiles, runs, measures and lays out a project, a folder or a single file.

```sh
cargo install --path crates/croft_cli

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
