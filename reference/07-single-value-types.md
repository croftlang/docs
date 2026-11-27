# Single-value types

A single-value type is a value around one value of another type. A score is an Int underneath, but as a `Score` it can't be mixed up with any other Int.

## Declaring one

Write the type it holds after a colon, with any constraints.

```croft
value Score: Int where Range(0, 999_999)
value Name: String where Length(1, 12)
```

## Making one

Make one by calling its name, or with a literal where one is expected.

```croft
let best: Score = 100
```

## Reading what it holds

Read what it holds with `.value`.

```croft
fn greet(name: Name): String = "Hello, " + name.value
```

## No borrowed operators

A single-value type has none of the operators of the type it holds. A `Score` has no `+`:

```croft
best + 10      // error: cannot do arithmetic on `Score`
```

So arithmetic makes a new one. Work with the value inside, then wrap the result. Making the new `Score` checks its constraints.

```croft
let better = Score(best.value + 10)
```
