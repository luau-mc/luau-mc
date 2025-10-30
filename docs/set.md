# `set` Library
The `set` library is a custom library exclusive to `luau-mc`.

Sets in `luau-mc` are custom mutable objects with a simple backing structure (`{ [any]: true }`).
Custom objects are used in favor of the much simpler hash map in order to guarantee the expected behavior from querying a set.
A plethora of functionality — much of which is syntax sugar — is provided to maximize user convenience.

## `set.create`
`<T...>( T... ) -> set<T...>`

Creates and returns a set with the given values.

```luau
local foo = set.create(1, 2, 3, 4)
print(foo:contains(1)) -- prints `true`
print(foo:contains(5)) -- prints `nil`
```

## `set.union`
`<T...>( ...set<T...> ) -> set<T...>`

Returns a new set that is the union of **all** of the given sets.

```luau
local foo = set.create(1, 2)
local bar = set.create(3, 4)
local baz = set.union(foo, bar) -- or `foo:union(bar)`
print(baz:contains(1)) -- prints `true`
print(baz:contains(3)) -- prints `true`
```

## `set.diff`
`<T...>( ...set<T...> ) -> set<T...>`

Returns a new set that is the difference of **all** of the given sets.

```luau
local foo = set.create(1, 2, 3, 4)
local bar = set.create(2, 3)
local baz = set.diff(foo, bar) -- or `foo:diff(bar)`
print(baz:contains(1)) -- prints `true`
print(baz:contains(2)) -- prints `false`
```

## `set.contains`
`<T...>( set<T...>, ...T ) -> boolean`

Returns a boolean indicating whether or not the given set contains **all** of the given values.

```luau
local foo = set.create(1, 2, 3, 4)
print(foo:contains(1))    -- prints `true`
print(foo:contains(0))    -- prints `false`
print(foo:contains(0, 1)) -- prints `false`
```

## `set.add`
`<T...>( set<T...>, ...T ) -> set<T...>`

Adds the given values to the given set (if they are not already present).
Returns the same set given as the first argument to support method chaining.

```luau
local foo = set.create(1, 2)
foo:add(3, 4)

print(foo:contains(1)) -- prints `true`
print(foo:contains(3)) -- prints `true`
```

## `set.remove`
`<T...>( set<T...>, ...T ) -> set<T...>`

Removes the given values from the given set (if they are present).
Returns the same set given as the first argument to support method chaining.

```luau
local foo = set.create(1, 2, 3, 4)
foo:remove(3, 4)

print(foo:contains(1)) -- prints `true`
print(foo:contains(3)) -- prints `false`
```

## `set.weak`
`<T...>( set<T...>, boolean ) -> set<T...>` | `<T...>( set<T...> ) -> ()`

1. If called with a boolean, the weak mode of the underlying table will be set, and the given set will be returned [to support method chaining]
2. Otherwise, if called without a boolean, a boolean representing the current weak mode state will be returned

Sets created with `set.create` initialize with weak mode set to `false`; sets created with `set.union` and `set.diff` inherit the weak mode state of the **first** given set.
Cloned sets inherit the weak mode state of the original set. 

```luau
local obj = {}
local foo = set.create(obj)

print(foo:weak()) -- prints `false`
print(#foo)       -- prints `1`

foo:weak(true)
obj = nil -- no more strong references exist, `obj` will be removed from `foo`

print(foo:weak()) -- prints `true`
print(#foo)       -- prints `0`
```

## `set.eq`
`( ...set<...any> ) -> boolean`

Returns a boolean indicating whether or not **all** of the given sets contain the exact same set of values.
That is to say, if a difference was made from the given sets, the output set would be identical to all of the input sets.

```luau
local foo = set.create(1, 2, 3, 4)
local bar = set.create(1, 2, 3, 4)
print(foo:eq(bar)) -- prints `true`

bar:remove(2, 3)
print(foo:eq(bar)) -- prints `false`
```

⚠️ The `==` operator should **not** be used to check if sets share the same contents, and should instead only be used to compare references.

## `set.len`
`<T...>( set<T...> ) -> number`

Returns the number of values in the set.
The `#` operator can also be used.

```luau
local foo = set.create(1, 2, 3, 4)
print(foo:len()) -- prints `4`
print(#foo)      -- prints `4`
```

## `set.clear`
`<T...>( set<T...> ) -> set<T...>`

Removes all values from the given set. Returns the same set given as the first argument to support method chaining.

```luau
local foo = set.create(1, 2, 3, 4)
print(foo:contains(1)) -- prints `true`

foo:clear()
print(foo:contains(1)) -- prints `false`
```

## `set.clone`
`<T...>( set<T...> ) -> set<T...>`

Returns a new copy of the given set.

```luau
local foo = set.create(1, 2, 3, 4)
local bar = foo:clone()
print(bar:contains(1)) -- prints `true`
print(bar:contains(5)) -- prints `false`
```

## `set.unpack`
`<T...>( set<T...> ) -> T...`

Returns the values in the set as a tuple.

```luau
local foo = set.create(1, 2, 3, 4)
print(foo:unpack()) -- prints `1 2 3 4`
```

## `set.toarray`
`<T...>( set<T...> ) -> { T }`

Returns the values in the set as an array.

```luau
local foo = set.create(1, 2, 3, 4)
local arr = foo:toarray() -- or `{ foo:unpack() }`
print(arr[1]) -- prints `1`
print(arr[5]) -- prints `nil`
```

## `set.raw`
`<T...>( set<T...> ) -> { [T]: true }`

Returns a copy of the underlying table.

```luau
local foo = set.create(1, 2, 3, 4)
local tbl = foo:raw()
print(tbl[1]) -- prints `true`
print(tbl[5]) -- prints `nil`
```

# Iteration
Sets can be iterated like any other table, though these offer slightly more elegant syntax:

```luau
for value in set do
    -- ...
end
```

# Using Type Annotations
Type annotations can be used to refine (or expand) the type signature of a set when the inference system is too strict/loose.
This is especially useful when working with complex sets or creating unions and differences.

```luau
local foo = set.create(1, 2, "Hello", "world!") :: set<...any>
local bar = set.create(1, 2, "Hello", "world!") :: set<number | string>

local foo = set.create(1, 2)
local bar = set.create("Hello", "world!")
local baz = set.union(foo, bar) :: set<number, string>
```

# Garbage Collection
Sets are automatically garbage collected when no more strong references to it are held.

# DEV
- I was spitballing the type signatures when I wrote this so I'm not actually sure if the type system would be happy with some of these signatures (for example things like `set.add` may need to become `<T...>( set<T...>, ...any ) -> set<T...>`)
- It is possible to revert back to an object-less structure by making tables returned by `set.create` immutable and changing their type signature to `{ [any]: true? }`, though this does not fix the fact that we bear the burden of making sure users understand that `false` is nowhere to be found despite the fact that that's how sets work in every other language
