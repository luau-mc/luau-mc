# `events` Library
⚠️ UNFINISHED

The `@minecraft/events` library exposes the game events provided by `Paper` and `Bukkit`.
These events are all implemented as `luau-mc` [signals]().

```luau
local events = require("@minecraft/events")
```

When events are fired, the first argument passed to connections will **always** be an [Event]() object.
The Event object is useful for [cancelling]() the event or [changing data about the event]().
**Mutable event data will always be contained by the Event object; immutable event data will always be passed as an argument to connections.**

## `BeaconToggled`
`signal<Event, Beacon, boolean>`

Fires whenever any [Beacon]() is activated or deactivated.
Passes the Beacon and a boolean represnting whether or not the Beacon is now active.

```luau
local events = require("@minecraft/events")
local BeaconActivated = events.BeaconToggled

BeaconToggled:connect(function( event, beacon, active )
    print(`Beacon at {beacon.position} is`, if active then "active" else "not active")
end)
```

‼️ **DEV** — This is a modified version of Paper's `BeaconActivated` and `BeaconDeactivated` events that has been consolidated into a singular `BeaconToggled` event

## `BellResonated`
`signal<Event, Bell, { LivingEntity }>`

Fires whenever any [Bell]() *resonates*.
A Bell resonates if Raiders are near the Bell after the Bell has been rung.
Passes the Bell that was rang and an array containing the Raiders that were "resonated".

As `BellRung` events can be [cancelled](), `BellResonated` will not fire until **after** `BellRung`.
This means that no `BellResonated` connection can run before any `BellRung` connection.
If `BellResonated` were going to fire but the corresponding `BellRung` event was cancelled, `BellResonated` will not fire at all.

```luau
local events = require("@minecraft/events")
local BellResonated = events.BellResonated

BellResonated:connect(function( event, bell, raiders )
    print(`Bell at {bell.position} rang and resonated {#raiders} raiders`)
end)
```

## `BellRung`
`signal<Event, Bell, Enum.Face, Entity?>`
`@cancellable`

Fires whenever any [Bell]() is rung.
Passes the Bell that was rang, the direction it was rang from, and the [Entity]() that rang it (if there is one).

`BellRung` events are [cancellable]().

```luau
local events = require("@minecraft/events")
local BellRung = events.BellRung

BellRung:connect(function( event, bell, direction, entity )
    if not event.cancelled then
        print(`Bell at {bell.position} was rang by {entity} facing {direction}`)
    end
end)
```

## `BlockBreakProgressChanged`
`signal<Event, Object, Block number>`

Fires each time the progress of breaking a block is changed.
Passes the [Object]() doing the breaking, the [Block]() being broken, and a number representing progress (`[0, 1]`).

```luau
local events = require("@minecraft/events")
local BlockBreakProgressChanged = events.BlockBreakProgressChanged

BlockBreakProgressChanged:connect(function( event, object, block, progress )
    print(`{object} is breaking {block.type} at {block.position} ({progress})`)
end)
```

## `BlockBrokeBlock`
`signal<Event, Block, Block>`
`@modifiable`

Fires whenever a [Block]() is broken as the result of another Block's action; for example, when a Piston is activated and extends into a space containing a Torch.
Passes the Block that caused the break and the Block that was broken.

`BlockBrokeBlock` events are [modifiable](): the amount of [EXP]() and the [ItemStacks]() that will be dropped can be changed.

```luau
local events = require("@minecraft/events")
local BlockBrokeBlock = events.BlockBrokeBlock

BlockBrokeBlock:connect(function( event, source, target )
    print(`{source.type} at {source.position} broke {target.type} and dropped {#drops} items`)
    print(`Will drop {event.exp} EXP and {#event.items} items`)
end)
```

## `BlockComposting`
`signal<Event, Block, ItemStack>`
`@cancellable`
`@modifiable`

Fires before a [Block]() is about to compost an item.
Passes the composting Block and the [items]() being composted.

`BlockComposting` events are [cancellable](), which will stop the items from being composted. They are also [modifiable](): you can change whether or not the compost level will rise.

```luau
local events = require("@minecraft/events")
local BlockComposting = events.BlockComposting

BlockComposting:connect(function( event, block, item )
    if event.cancelled then
        return
    end

    print(`{block.type} at {block.position} is going to compost {items.type}`)
    event.raiseLevel = true
end)
```

‼️ **DEV** — This is a modified version of Paper's `CompostItemEvent` which allows the user to cancel the event from this signal rather than detouring through the `InventoryMoveItemEvent` equivalent

## `BlockDispenseFailed`
`signal<Event, Block>`
`@modifiable`

Fires whenever a [Block]() fails to dispense an item because its inventory is empty.
Passes the dispensing Block.

`BlockDispenseFailed` events are [modifiable](): you can change whether or not to play the effects when an empty dispenser fires.

```luau
local events = require("@minecraft/events")
local BlockDispenseFailed = events.BlockDispenseFailed

BlockDispenseFailed:connect(function( event, block )
    print(`{block.type} at {block.position} failed to dispense`)
    event.playEffects = false
end)
```

## `BlockDispensing`
`signal<Event, Block, number>`
`@cancellable`

Fires before a [Block]() is about to dispense an item.
Passes the dispensing Block and the slot containing the item that will be dispensed.

`BlockDispensing` events are [cancellable](), which will stop the item from being dispensed.

```luau
local events = require("@minecraft/events")
local BlockDispensing = events.BlockDispensing

BlockDispensing:connect(function( event, block, slot )
    if event.cancelled then
        return
    end

    local inv = block.inventory
    local item = inv[slot] :: ItemStack
    
    print(`{block.type} at {block.position} is going to dispense {item.type}`)
end)
```

## `BlockShearing`
`signal<Event, Object, Block, ItemStack, { ItemStack }>`
`@cancellable`

Fires before a [Block]() is sheared (not destroyed, but sheared) by an [Object]().
Passes the Object using the shearing item, the Block that will be sheared, the [item]() being used to shear the block, and an array containing the [items]() that will be dropped.

`BlockShearing` events are [cancellable](), which will stop the block from being sheared.

Today, only `Players` can shear blocks, meaning that, for right now, `Object` will only ever be a `Player`.
Despite knowing this, we use `Object` in an effort to futureproof.
Should Minecraft ever allow a mob to shear blocks, or blocks to shear blocks, or anything else, it'll be unlikely that the structure of this signal will need to change, giving you less work to do.
With that in mind, if you want to better futureproof your code, you should always add logic to check that `Object` is a `Player`, even if it's redundant to do so right now.

```luau
local events = require("@minecraft/events")
local BlockShearing = events.BlockShearing

BlockShearing:connect(function( event, object, block, item, drops )
    if not event.cancelled then
        print(`{object} is going to shear {block.type} at {block.position} using {item.type} which will drop {#drops} items`)
    end
end)
```

‼️ **DEV** — This is a modified version of Paper's `PlayerShearBlockEvent` which has been generalized to take in an `Object` instead of a `Player` in an attempt to futureproof

# DEV
- Generalizing signals may actually not be such a good idea. Though futureproofing is a good thing to target, we must consider that users often seek specific actions and they do not want to analyze the data from a generic event to know if it's something they're actually interested in. It's also inherently less performant. The trade-off between aesthetics/API surface area and user convenience/performance must be better weighed.
