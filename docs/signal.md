# `signal` Library
The `signal` library is a custom library exclusive to `luau-mc`.

Signals are one of the most powerful development tools and are used frequently throughout `luau-mc`.
Signals make it trivial to bind logic to events. These events are often controlled by external systems: signals provide an inherently decoupled interface for said events.

The motivation behind this library is to provide users with a unified implementation as to avoid a situation like ROBLOX's where there exists more third-party implementations than can be named.

## `signal.create`
`() -> signal<>`

Creates and returns a new signal object.

```luau
local sig = signal.create()
```

Signal objects re-expose the `signal` library functions as methods via the [colon-call syntax sugar]().

## `signal.fire`
`( signal<T...>, T... ) -> ()`

Fires the given signal, passing the given arguments to connected callbacks.

```luau
local sig = signal.create()
sig:fire("Hello world!")
```

## `signal.destroy`
`( signal<T...> ) -> ()`

Destroys the given signal, which will:
1. Put the object into an unusable state
2. Disconnect any connections
3. Resume any yielded threads

```luau
local sig = signal.create()
sig:destroy()
sig:fire() -- ✖️ — errors
sig:wait() -- ✖️ — errors
```

⚠️ The object itself cannot be removed (and thus the memory cannot be freed up) until all remaining strong references are removed.

⚠️ Signals will be automatically destroyed [and GCed] if no strong references exist, even if `signal.destroy` is not called, and even if connections are connected and/or threads are yielded because of `signal.wait` calls.

## `signal.connect`
`( signal<T...>, ( T... ) -> () ) -> userdata`

Connects the given callback to the given signal and returns a connection identifier.
The given callback will be called each time the signal is fired.

```luau
local sig = signal.create()
sig:connect(print)
sig:fire("Hello world!") -- `Hello world!` is printed to the console
```

Connections can be disconnected at any time using `signal.disconnect`.
Connections remain connected until the signal is destroyed or the connection is manually disconnected, whichever comes first.

## `signal.once`
`( signal<T...>, ( T... ) -> () ) -> userdata`

Creates a connection that will automatically disconnect itself the next time the signal is fired.
Returns a connection identifier.

```luau
local sig = signal.create()
sig:once(print)
sig:fire("Hello world!") -- `Hello world!` is printed to the console
sig:fire("Hello world!") -- nothing is printed because the `print` connection automatically disconnected itself
```

## `signal.wait`
`( signal<T...> ) -> T...`

Yields the current thread until the next time the signal is fired.

```luau
local sig = signal.create()
sig:wait() -- this thread is now indefinitely yielded
print("Hello world!") -- this does not print yet

-- on another thread
sig:fire() -- `Hello world!` is printed because the yielded thread was resumed
```

Arguments passed to connections are returned when the thread is resumed:

```luau
local sig = signal.create()
local str = sig:wait()
print(str) -- again, this does not print yet

-- on another thread
sig:fire("Hello world!") -- `Hello world!` is printed because the yielded thread was resumed and the argument was returned
```

⚠️ Unlike how connections can be disconnected at any point, calls to `signal.wait` **cannot** be "prematurely" abandoned (because the thread is immediately yielded and thus there is no way to retrieve the underlying connection).

⚠️ Any threads still yielded when a signal is destroyed or otherwise GCed will be resumed with no values.

## `signal.connected`
`( signal<T...>, userdata ) -> boolean`

Returns a boolean indicating whether or not the given connection is still connected.

```luau
local sig = signal.create()
local cnx = signal:connect(function() end)
print(sig:connected(cnx)) -- prints `true`
```

⚠️ If a connection identifier is given along with a signal that did **not** create the connection, an error will be thrown.

## `signal.disconnect`
`( signal<T...>, userdata ) -> ()`

Disconnects the given connection, preventing the underlying callback from being called by subsequent `signal.fire` calls.

```luau
local sig = signal.create()
local cnx = signal:connect(print)

signal:fire("Hello world!") -- `Hello world!` is printed
signal:disconnect(cnx)
signal:fire("Hello world!") -- nothing is printed because the `print` connection was disconnected
```

⚠️ If a connection identifier is given along with a signal that did **not** create the connection, an error will be thrown.

## `signal.public`
`( signal<T...> ) -> signal<T...>`

Returns a separate, modified version of the given signal that **cannot** be fired or destroyed.
This is useful if you want to expose a signal but want to make sure that external systems cannot fire it or destroy it.

```luau
local sig = signal.create()
sig:fire() -- ✔️

sig = sig:public()
sig:fire()    -- ✖️ — errors
sig:destroy() -- ✖️ — errors
sig:wait()    -- ✔️
```

# Using Type Annotations
Type annotations can be used to make working with signals even easier. By annotating a signal, the arguments passed from `signal.fire` calls to connected callbacks are known. To demonstrate:

```luau
local sig = signal.create() :: signal<string, number>

sig:connect(function( foo, bar )
    -- `foo` is `string`
    -- `bar` is `number`
end)

sig:fire("Hello world!", 123) -- ✔️
sig:fire(123, "Hello world!") -- ✖️ — the first argument should be `string`, and the second argument should be `number`
sig:fire("Hello world!")      -- ✖️ — 2 arguments are expected but only 1 is given
sig:fire()                    -- ✖️ — 2 arguments are expected but none are given
```

# DEV
- Notes regarding recursive firing behavior are needed
- Notes regarding whether or not firing signals is a yielding action are needed
