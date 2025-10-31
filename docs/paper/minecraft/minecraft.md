# `minecraft` Directory
Most `Paper`/`Bukkit` functionality is exposed through libraries stored under the `minecraft` directory.
These libraries exist under the `minecraft` directory as to avoid clouding the global environment with frequently used terms (ie. `events`): we let users decide what they want to name these libraries when they import them.

The `minecraft` directory is just like a folder in the project's file structure.
```luau
local events = require("@minecraft/events")
local entities = require("@minecraft/entities")
```

## `event` Library
The `minecraft/event` library exposes all of the events (signals) offered by `Paper` and `Bukkit`.

```luau
local events = require("@minecraft/events")
events.BeaconActivated:connect(...)
events.BeaconDeactivated:connect(...)
```

[See the full `event` library here.]()
