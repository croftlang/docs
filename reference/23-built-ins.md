# Built-ins

These functions and methods are in every program, with no import. Anything else from the standard library comes from a [standard context](24-standard-contexts.md).

## Functions

`Some` and `None` make options, `Ok` and `Err` make results, and `Range` and `Length` are [constraints](09-constraints.md).

```croft
Some(value)   None   Ok(value)   Err(error)
Range(n, low, high)   Length(items, low, high)
```

## Strings and numbers

```croft
name.length   name.is_empty   "dun" + "geon"   score.to_string()
```

## Lists

Reading a list. `first` and `last` give back an `Option`, since the list might be empty.

```croft
items.length   items.is_empty   items.first   items.last
items.get(0)   items.contains(item)   items.index_of(item)
```

Making a new list from an old one.

```croft
items.add(item)   items.drop(1)   items.take(3)   items.reverse()
items + more   [first] + rest
```

`unique` keeps the first of each item, in order.

```croft
items.unique()   // [3, 1, 3].unique() is [3, 1]
```

`insert` puts an item at a place from 0 to the list's length, and moves the items from there on one place later.

```croft
items.insert(0, item)
```

A place known to be below 0 is a `croft check` error. One past either end stops the program, unless the target sets `checks = false`, when the item goes at the nearest end.

Methods that take a lambda call it with each item.

```croft
items.filter(i -> i.ready)   items.map(i -> i.name)   items.find(i -> i.ready)
items.all(i -> i.ready)   items.any(i -> i.ready)   items.count(i -> i.ready)
items.sum(i -> i.points)   items.min_by(i -> i.cost)   items.max_by(i -> i.cost)
```

`fold` builds a total from each item in turn. Its lambda takes two parameters, the total so far and the item, and gives back the next total. The total starts as the first argument.

```croft
items.fold(0, (total, i) -> total + i.points)
```

## Sorting

`sort` puts numbers in order by size, and strings byte by byte. `sort_by` orders by the whole number its lambda gives. Items with the same key keep their order.

```croft
scores.sort()   names.sort()   items.sort_by(i -> i.cost)
```

## Lists of entities

A list of [entities](10-aggregates-and-entities.md) can also be looked up, replaced and removed by identity.

```croft
enemies.get(enemy_id)   enemies.contains(enemy_id)
enemies.replace(enemy)   enemies.remove(enemy_id)
```

## Options

```croft
found.is_some   found.is_none   found.map(e -> e.id)   found.unwrap_or(0)
```

## Results

```croft
loaded.is_ok   loaded.is_err   loaded.map(r -> r.name)   loaded.unwrap_or(fallback)
loaded.map_err(e -> GameError.SaveDamaged)
```
