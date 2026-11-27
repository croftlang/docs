# Standard contexts

The standard library is a set of contexts you import from, like any other. Each context says what it relies on, and the library can grow without filling every namespace.

## What needs no import

Two standard contexts are always there. Std.Console has the `Console`, `Arguments`, `Environment` and `Process` [ports](25-ports.md), and Std.Storage is for [repositories](26-repositories.md).

Everything else beyond the [built-ins](23-built-ins.md) is imported.

## Importing

An import covers every file of the context. A standard context needs no arrow in the domain's `contexts`.

```croft
context Notes

use { trim, split, to_upper, to_bytes, to_text, replace, join, code_at, from_code, to_hex, from_hex, to_decimal } from Std.Text
use { Decimal, max, min, abs, clamp, pow, sign, checked_add } from Std.Numbers
use { Files, FileError } from Std.Files
```

Importing costs nothing you don't use: whatever is imported, the generated C only carries the runtime functions the program calls.

Leave an import out and the compiler tells you which one you need.

```croft
// `FileError` needs `use { FileError } from Std.Files`
// `Files` needs `calls { Files } from Std.Files`
```

## Std.Console

Printing, to the normal output or the error output.

```croft
Console.print("hello")   Console.print_error("no such room")
```

Reading a line of input. It gives `None` at the end of the input.

```croft
Console.read_line()   // Option<String>
```

The program's arguments, and its environment variables. `Environment.get` gives `None` when the variable isn't set.

```croft
Arguments.all()   // List<String>
Environment.get("HOME")   // Option<String>
```

Stopping the program with an exit code from 0 to 255.

```croft
Process.exit(1)
```

## Std.Text

Strings, and text to and from bytes.

Searching a string. `index_of` gives back an `Option<Int>`.

```croft
name.contains("Sir")   name.starts_with("Sir")   name.ends_with("!")
name.index_of("Sir")   // Option<Int>
```

Changing case, and trimming. `trim` takes off the spaces, tabs, `"\r"` and `"\n"` at either end.

```croft
name.to_upper()   name.to_lower()   name.trim()
```

Cutting a string up.

```croft
name.slice(0, 3)   line.split(",")
```

`lines` leaves out each `"\n"` or `"\r\n"`, and gives no empty line after the last.

```croft
text.lines()
```

Building strings. `join` is for a `List<String>`.

```croft
html.replace("&", "&amp;")   "-".repeat(8)
score.to_string().pad_left(5, "0")   name.pad_right(10, " ")
names.join(", ")
```

Turning text into bytes and back.

```croft
"DN".to_bytes()   data.to_text()
```

## Text to numbers and back

`to_int` reads digits with an optional `"-"`, and gives `None` past `Int`'s range.

```croft
text.to_int()   // Option<Int>
```

`to_decimal` reads a [`Decimal`](20-numbers.md). The number of places comes from where it goes.

```croft
let price: Option<Decimal<2>> = "12.5".to_decimal()   // Some(12.50)
```

`code_at` gives a character's code from 0 to 255, or `None` past the end. `from_code` goes the other way, and a code that isn't from 0 to 255 is a mistake.

```croft
name.code_at(0)   // Option<Int>
from_code(65)   // "A"
```

`to_hex` writes capitals with no prefix, and a number below zero as its 32 bits. `from_hex` reads either case, with no prefix or sign, and gives `None` past 32 bits.

```croft
255.to_hex()   // "FF"; (-1).to_hex() is "FFFFFFFF"
from_hex("ff")   // Some(255)
```

## Std.Numbers

`max`, `min` and `abs`, and the `Decimal` type.

```croft
max(a, b)   min(a, b)   abs(n)   Decimal<2>
```

`clamp` keeps a number between a low and a high. A low above the high is a mistake.

```croft
clamp(volume, 0, 11)   // volume kept from 0 to 11
```

`pow` raises a number to a power. An exponent below zero, or an answer past 32 bits, is a mistake.

```croft
pow(2, 10)   // 1024
```

`sign` gives -1, 0 or 1.

```croft
sign(n)
```

The checked versions give `None` when the answer goes past 32 bits. [Numbers](20-numbers.md) says what happens to the mistakes above.

```croft
checked_add(a, b)   checked_subtract(a, b)   checked_multiply(a, b)   // Option<Int>
```

## Std.Collections

`Map<K, V>` and `Set<T>` hold as many entries as their `Length` constraints allow.

```croft
use { Map, Set } from Std.Collections
```

A map's `get` gives back an `Option`. `set` and `remove` give back a changed copy.

```croft
let stock: Map<String, Int> = ["sword": 1, "key": 2]
stock.get("key")            // Some(2)
stock.set("map", 1)         // a copy with map added
stock.remove("sword")       // a copy without sword
```

The rest read the map.

```croft
stock.contains("key")   stock.length   stock.is_empty   stock.keys   stock.values
```

A set is written like a list, or `[]` when empty. An item listed twice is a mistake.

```croft
let seen: Set<Int> = [3, 1]
```

`add` and `remove` give back a changed copy. Adding an item that is already there changes nothing.

```croft
seen.add(2)                 // a copy with 2 added; adding 3 again changes nothing
seen.remove(3)              // a copy without 3
```

`items` gives the items in the order they were added.

```croft
seen.contains(1)   seen.length   seen.is_empty
seen.items                  // List<Int>: [3, 1, 2]
```

## Std.Encoding

Bytes as text and back, text for URLs, and checksums.

```croft
use { to_hex, from_hex, to_base64, from_base64, percent_encode, percent_decode, crc16, crc32 } from Std.Encoding
```

The `to_` names, the percent methods, `crc16` and `crc32` are methods of what they read, as `to_bytes` is in Std.Text. `to_hex`, `to_base64`, `crc16` and `crc32` belong to `Bytes`, and the percent methods to a `String`. `from_hex` and `from_base64` are functions taking the text, since they make `Bytes` from a `String`.

### Hex

`to_hex` writes two upper-case digits for each byte. `from_hex` reads either case, two digits for each byte, with nothing between them.

```croft
let header: Bytes = [0x44, 0x4E, 0xFF]
header.to_hex()            // "444EFF"
from_hex("444eff")         // Some([0x44, 0x4E, 0xFF]); None for "444", "0x44" or "44 4E"
```

### Base64

Base64 is RFC 4648's standard alphabet with `=` padding.

```croft
header.to_base64()         // "RE7/"
from_base64("RE7/")        // Some([0x44, 0x4E, 0xFF]); None for anything but base64
```

`from_base64` gives `None` for a length that isn't a multiple of 4, a character outside the alphabet, `=` anywhere but the end, or leftover bits that aren't 0. So each `Bytes` has exactly one base64 text.

### Percent encoding

`percent_encode` keeps letters, digits and `-._~`, RFC 3986's unreserved characters. It writes every other character as `%` and two upper-case hex digits, each byte of a character on its own.

`percent_decode` reads either case, gives `None` for a `%` not followed by two hex digits, and leaves `+` as it is.

```croft
"fish & chips".percent_encode()     // "fish%20%26%20chips"
"fish%20%26%20chips".percent_decode()   // Some("fish & chips"); None for "100%"
```

### Checksums

`crc16` is CRC-16/CCITT-FALSE: polynomial 0x1021, starting from 0xFFFF. `crc32` is the CRC-32 of zip, PNG and Ethernet. Its 32 bits are given as an `Int`, so one with the top bit set is below zero.

Text is checked through its bytes.

```croft
"123456789".to_bytes().crc16()   // 0x29B1
"123456789".to_bytes().crc32()   // 0xCBF43926 as 32 bits: -873187034
```

### How long the results are

Each result has room for all of it: two characters for each byte of hex, four for each three bytes of base64, and three for each character percent-encoded. That goes up to what a string or `Bytes` with no `Length` holds. A longer result stops the program, or is cut short when the target sets `checks = false`, as a string is.

### Hex in two places

`to_hex` and `from_hex` are in both Std.Text and Std.Encoding.

`to_hex` is a method, so the value it is called on picks one: `255.to_hex()` is Std.Text's and `header.to_hex()` is Std.Encoding's. A context that uses both imports `to_hex` from each.

`from_hex` is a function, so the import decides. From Std.Text it gives an `Option<Int>`, and from Std.Encoding an `Option<Bytes>`. A context imports it from one of them. Importing it from both is an error:

```croft
// `from_hex` can't come from both `Std.Text` and `Std.Encoding`: they mean different things, so import it from one
```

Two contexts of one program can each import their own. That is why the Std.Encoding samples above belong in a context of their own, such as `context Packets`: the Notes context already takes `from_hex` from Std.Text.

## Written in Croft

Std.Files, Std.Http, Std.Random and Std.Time are written in Croft. Each is a port, its errors, and an [adapter](28-adapters.md) that croft.toml can bind:

- `StdioFiles`, on any machine with C's stdio
- `CurlHttp`, wherever libcurl is, which also brings HTTPS
- `SeededRandom`, on any machine
- `SystemClock`, wherever C's `time` and `localtime` are

The [project file](30-project-file.md) names an adapter for each port.

```toml
[target.desktop.adapters]
Files = "StdioFiles"
Http = "CurlHttp"          # link with -lcurl
Random = "SeededRandom"
Clock = "SystemClock"
```

## Std.Files

The `Files` port has `read`, `read_part`, `size`, `write`, `append`, `exists`, `remove`, `list`, `make_folder`, `copy` and `rename`.

```croft
service Keep(): Result<Unit, FileError> {
    calls { Files }
} = {
    Files.write("notes.txt", "first line".to_bytes())?
    let data = Files.read("notes.txt", 4096)?
    ...
}
```

`read` takes at most the number of bytes you give it, and a longer file fails with `NoRoom`. `read_part` skips some bytes and reads up to a number more, giving none past the end. `size` works however large the file is, up to 2147483647.

```croft
let data = Files.read("notes.txt", 4096)?   // at most 4096 bytes
let header = Files.read_part("save.bin", 16, 8)?   // up to 8 bytes, skipping the first 16
let bytes = Files.size("logo.png")?
```

`list` gives the names in a folder in byte order, with a `/` on the end of each folder's name.

```croft
let pages = Files.list("docs")?   // ["a.md", "b.md", "drafts/"], in byte order
```

`make_folder` also makes each folder on the way. `copy` works for a file of any size, not only what a `Bytes` holds. `rename` moves a file on the same disk.

```croft
Files.make_folder("site/blog")?    // and site, if it isn't there yet
Files.copy("logo.png", "site/logo.png")?
Files.rename("notes.txt", "site/notes.txt")?
```

## Std.Http

The `Http` port has four operations. Each gives back a `Response { status, headers, body }` or an `HttpError`.

```croft
// get(url, headers)   post(url, headers, body)   put(url, headers, body)   delete(url, headers)
```

Instead of a `use`, a service can bring the port in with `calls ... from`. Headers are a map, or `[:]` for none.

```croft
use { Response, HttpError } from Std.Http

service Upload(save: Bytes): Result<Unit, HttpError> {
    calls { Http } from Std.Http
} = {
    let response = Http.post("https://example.com/saves", ["Content-Type": "application/octet-stream"], save)?
    let list = Http.get("https://example.com/saves", [:])?   // no headers
    ...
}
```

The body holds at most the target's `bytes_length`. A longer one fails with `TooLarge`.

### What CurlHttp does

`CurlHttp` speaks only http and https, redirects included. It follows at most 10 redirects and gives the last answer's status, headers and body. Any other URL, such as `file:///etc/passwd`, fails with `Unreachable`, and so does an eleventh redirect.

Each request takes a URL of up to 1024 characters. The URL is borrowed for the call, so no other string is made as long.

A request takes up to 8 headers, which `CurlHttp` sends with every redirect. It leaves out `Authorization` and `Cookie` when a redirect leads to another host.

A header fails the request with `Unreachable` before anything is sent if its name or value holds a carriage return, a line feed or a NUL, or if its name is empty or holds a colon.

## Std.Random

`between(low, high)` gives a whole number with both ends included.

```croft
use { Random } from Std.Random

service Roll(): Int {
    calls { Random }
} = Random.between(1, 6)   // `Random.between(6, 1)` doesn't compile: it breaks `low <= high`
```

`SeededRandom` is a xorshift32 generator. It is seeded once from C's `time` the first time it is used, so each run gives different numbers, and a machine without a clock gives the same ones every run. It is fine for games and samples, and never for anything secret.

## Std.Time

Std.Time has `Date { year, month, day }`, `Time { hour, minute, second }`, and the `Clock` port.

```croft
use { Date, Time, Clock, DayInMonth, is_leap_year, days_between, add_days } from Std.Time
```

The clock gives today's date and the time now.

```croft
service Due(): Int {
    calls { Clock }
} = {
    let today = Clock.today()   // Date { year: 2026, month: 11, day: 27 }
    let now = Clock.now()       // Time { hour: 9, minute: 30, second: 0 }
    ...
}
```

`SystemClock` reads the machine's own time zone. A machine with no clock gives 1 January 1970 at midnight.

### Dates

A `Date` is a day on the Gregorian calendar, carried back before 1582, from year 1 to 9999.

A date is only ever one its month has. Its invariant `DayInMonth` makes a 29 February in 2025 an error from `croft check`, and with checks on, such a date made while the program runs stops it.

```croft
// `Date { year: 2025, month: 2, day: 29 }` doesn't compile: it breaks `DayInMonth`
```

`days_between` and `add_days` count leap years. `days_between` is negative when the second date is earlier.

```croft
let due = add_days(today, 30)   // Date { year: 2026, month: 12, day: 27 }
add_days(Date { year: 2024, month: 2, day: 28 }, 1)   // 29 February 2024, a leap year
days_between(today, Date { year: 2027, month: 1, day: 1 })   // 35
```

A date past either end is an error from `croft check` when the date and the days are known. Otherwise it stops the program with checks on.

### Building on the calendar

`DayInMonth` and `is_leap_year` are published, so a program's own [constraints](09-constraints.md) can build on them. `is_leap_year` is true of a year 4 divides, unless 100 does and 400 doesn't: `is_leap_year(1900)` is false, and `is_leap_year(2000)` is true.

```croft
constraint LeapYear(year: Int) = is_leap_year(year)

value Games {
    year: Int where LeapYear   // a `Games` given 2025 for `year` doesn't compile: it breaks `LeapYear`
    opening: Date
}
```
