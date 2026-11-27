# Translations

A translation is the anti-corruption layer. It maps one context's term to another's, so neither leaks into the other.

## Writing one

A translation is written like a [function](19-functions.md): it takes the other context's term and gives back your own. Here Inventory turns Dungeon's treasure into one of its own items.

```croft
// in context Inventory
translation TreasureToItem(treasure: Dungeon.Treasure): Item =
    match treasure.kind {
        Coins(count) -> Item.Gold(amount: count)
        Flask -> Item.Potion(heal: 20)
        Key(door) -> Item.Key(door: door)
    }
```

Dungeon still talks about treasure and Inventory still talks about items. Only the translation needs to know both.
