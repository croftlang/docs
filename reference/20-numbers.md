# Numbers

Croft has whole numbers, exact decimals and bytes. A sum gives the same answer on every machine, and stays small and quick on an 8-bit one.

## Int

`Int` is a whole number of exactly 32 bits on every machine, from -2147483648 to 2147483647.

Where the compiler knows which values an `Int` can hold, it keeps it in the smallest type that holds them. It knows from a `Range` on a field, parameter or loop counter, from a length, or from arithmetic on those.

When every value fits in 16 bits, the arithmetic is done in 16 bits, which is what 8-bit machines do fastest. The answer is always the one 32 bits would give.

## Dividing

`/` rounds toward zero, and `%` has the sign of the number divided.

```croft
// half(-7) is -3, and rest(-7) is -1
fn half(n: Int): Int = n / 2

fn rest(n: Int): Int = n % 2
```

## When the answer doesn't fit

Arithmetic with no answer in 32 bits is a mistake. That means:

- `+`, `-` or `*` going past 32 bits
- `-` of the lowest number, or dividing it by -1
- `/` or `%` by zero

Where the values are known, `croft check` reports it, through calls to functions as well.

```croft
fn grow(n: Int): Int = n * 1000

// an error: `grow(5000000)` works out `5000000 * 1000`, which gives 5000000000
fn grown(): Int = grow(5000000)
```

Otherwise the program stops with "a whole number doesn't fit in 32 bits" or "division by zero".

A target with `checks = false` wraps the answer around in 32 bits instead, and dividing by zero gives 0.

## Decimal

`Decimal<2>` is a fixed-point number with two decimal places. It is exact, and works on machines without floating point. It comes from Std.Numbers.

```croft
import { Decimal } from Std.Numbers
```

It keeps a whole number of hundredths in the same 32 bits, so it holds up to 21474836.47. An answer past that is a mistake in the same way as for `Int`, and stops the program with "a decimal doesn't fit in 32 bits".

A literal such as `2.50` is a `Decimal` wherever one is expected, and an `Int` works anywhere a `Decimal` does.

```croft
fn with_tax(price: Decimal<2> where Range(0, 999.99)): Decimal<2> = price * 1.20
```

Adding and subtracting are exact. Multiplying and dividing round half to even.

```croft
// 10.00 / 3 is 3.33; 0.05 / 2 rounds to 0.02, the even neighbour
fn share(total: Decimal<2>, people: Int): Decimal<2> = total / people
```

`to_string` shows both places, and `round` gives back an `Int`.

```croft
let price: Decimal<2> = 4
price.to_string()  // "4.00"
price.round()      // 4, an Int
```

That sample also shows that a `let` can name its type.

## More from Std.Numbers

Std.Numbers also has `clamp`, `pow` and `sign`, listed with the [standard contexts](24-standard-contexts.md).

A `pow` past 32 bits is a mistake, as `*` is. So are an exponent below zero and a `clamp` whose low is above its high. `croft check` reports them when the values are known, and otherwise the program stops. With `checks = false`, `pow` gives 0 for an exponent below zero, and `clamp` gives its low.

`checked_add`, `checked_subtract` and `checked_multiply` give an `Option<Int>` instead: `None` when the answer doesn't fit in 32 bits, whatever the checks.

## Bits

Numbers can be written in hex or binary, and `_` can group the digits.

```croft
const FIRE: Int = 1 << 4
const MASK: Int = 0b0001_1111
const SCREEN: Int = 0x4000
```

Bit operators work on `Int` and bind tighter than comparisons, so `keys & FIRE != 0` tests the bit, as you would hope.

```croft
fn fire_pressed(keys: Int): Bool = keys & FIRE != 0

fn flip(keys: Int): Int = ~keys & MASK
```

Shifts go from 0 to 31.

## Bytes

`Bytes` holds values from 0 to 255, written as a list of numbers.

```croft
let header: Bytes = [0x44, 0x4E, 1]
```

It has `length`, `get`, `set`, `add` and `slice`, and `+` joins two together.

`add_u16` and `get_u16` write and read a number in two bytes, the low byte first.

```croft
let tape = header.add_u16(6912)        // 0x1B00 as 00 1B, low byte first
tape.get_u16(3)                        // Some(6912)
```

For machines that keep the high byte first, such as the 68000, use `get_u16_be` and `add_u16_be`.

## How much a Bytes holds

A `Bytes` with a `Length` keeps room for that many values, however many that is.

One without, such as a field, a `let` that `set` changes, or an HTTP response's body, keeps room for the target's `bytes_length`. That is 64 unless [croft.toml](30-project-file.md) says otherwise, so one long `Length` doesn't make every `Bytes` as long.

A longer value going into one stops the program, or is cut short when the target sets `checks = false`.

A `Bytes` parameter borrows what it is given, so it takes one of any length.

[Std.Encoding](24-standard-contexts.md) writes `Bytes` as hex or base64 and reads them back, and gives their CRC-16 or CRC-32.
