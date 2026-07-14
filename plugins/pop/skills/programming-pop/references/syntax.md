# Pop Lang Syntax

Use this reference when writing or reviewing `.pop` source. It records the
accepted source shape and deliberately avoids unstable library APIs.

## Contents

- [Source files and names](#source-files-and-names)
- [Types and literals](#types-and-literals)
- [Declarations](#declarations)
- [Functions and generics](#functions-and-generics)
- [Control flow](#control-flow)
- [Optional values and results](#optional-values-and-results)
- [Collections](#collections)
- [Compile time, attributes, and documentation](#compile-time-attributes-and-documentation)
- [Formatting](#formatting)

## Source Files and Names

Each `.pop` Module has one file-scoped namespace, followed by `using` directives
and declarations. Neither header has an `end`.

```luau
namespace Studio.Gameplay

using Studio.Data
using Physics = Studio.Simulation.Physics

private const INITIAL_SCORE = 0
```

`using` is static name resolution. It does not load code, create a dependency,
export names, or re-export declarations.

| Entity | Form |
| --- | --- |
| Namespace, Package, Bubble, type, case, attribute | `PascalCase` |
| Function, method, field, local, parameter, Module file | `camelCase` |
| Constant | `UPPER_SNAKE_CASE` |
| Intentionally ignored binding | `_` |

Do not write lowercase `snake_case`. Treat acronyms as words: `HttpRequest`,
`parseJson`, `userId`, `Utf8`.

Namespace-scope declarations accept `public`, `internal`, or `private`. Omitted
visibility means `internal`, except an omitted binary-root `main`, which is
private. Namespace declarations have no visibility. Interface members need no
redundant modifier.

## Types and Literals

Built-in names use PascalCase. Important accepted forms include `Boolean`,
`String`, `Byte`, `Int`, explicit signed/unsigned integer widths, `Float32`,
`Float64`, and `Never`. `Float` is the accepted `Float64` spelling.

```luau
local enabled: Boolean = true
local count: Int = 42
local ratio: Float64 = 1.5
local name: String = "Pop"
local missing: String? = nil
```

Integer and floating values do not widen implicitly. Convert with a target-type
call; checked conversions may trap:

```luau
local ratio = Float64(count)
local narrowed = Int32(total)
```

Strings are immutable valid UTF-8. Concatenate with `..`; interpolate with
backticks and `{expression}`. Initial interpolation accepts only strings,
booleans, and primitive numbers.

```luau
local path = "src" .. "/main.pop"
local message = `checked {count} files at {path}`
local exact = String(count)
```

Do not use JavaScript `${value}`, universal `toString`, implicit formatting, or
ambient locale conversion.

Function types retain Luau direction:

```luau
private type Predicate = function(value: Int): Boolean
```

`T?` is `T | nil`. Parenthesized comma types represent fixed type packs/tuples.

## Declarations

Public declarations require complete checked XML documentation. Declaration
snippets in this reference sometimes omit repeated documentation so the syntax
being demonstrated stays visible; add the required documentation in real public
APIs and use the complete example in the documentation section as a starting
point.

### Records

Use records for ordinary typed data. Update them with `with` rather than class
setters or copy builders.

```luau
public record Player
    name: String
    score: Int = 0
end

public function award(player: Player, points: Int): Player
    return player with {
        score = player.score + points,
    }
end
```

### Unions and Errors

Use unions for closed alternatives and `error` for nominal recoverable error
families.

```luau
public union LoadState
    Idle
    Loading(progress: Float)
    Ready(data: Bytes)
end

public error LoadError
    Missing(path: String)
    InvalidData(message: String)
end
```

Cases are qualified: `LoadState.Idle`, `LoadError.InvalidData("...")`.

### Enums and Aliases

Enums are nominal, payload-free, and not integers. Initial type aliases are
non-generic and erase to their target type.

```luau
public enum Color
    Red
    Green
    Blue
end

public type Scores = {[String]: Int}
```

### Interfaces and Classes

Interfaces are nominal and small. Classes explicitly state `implements` and use
Luau colon methods. Prefer records and functions unless identity or mutable
lifecycle is essential.

```luau
public interface Reader
    function read(count: Int): String
end

public class FileReader implements Reader
    private closed: Boolean = false

    public function FileReader:read(count: Int): String
        return ""
    end

    public function FileReader:close()
        self.closed = true
    end
end
```

There is no duck typing, marker-interface convention, universal base object, or
runtime member lookup.

## Functions and Generics

Every parameter is explicitly typed. A missing result annotation means the
explicit empty result pack (`void`), not inference.

```luau
public function add(left: Int, right: Int): Int
    return left + right
end

private function log(message: String)
    print(message)
end
```

Local functions and closures use ordinary `function ... end` syntax:

```luau
local offset = 3
local addOffset = function(value: Int): Int
    return value + offset
end
```

Generic declarations use `<T>`. A normal call to a directly resolved generic
namespace function, static method, or union/error case may infer the complete
type-argument list when arguments, expected result, and bounds yield one unique
static solution. Use `<<...>>` to supply every argument explicitly; partial
explicit lists are not supported.

```luau
private function first<T>(values: {T}): T?
    return values[1]
end

local name = first<<String>>(names)

private function consume<T, TSource: Iterable<T>>(source: TSource)
end
```

A type parameter accepts at most one nominal interface bound after `:`. Bounds
may mention earlier parameters only. Inference failure is a static diagnostic;
it never guesses a default type or defers generic selection to runtime.

Multiple returns are fixed packs. Pop neither pads missing results with `nil`
nor silently discards extras.

```luau
private function divide(value: Int, divisor: Int): (Int, Int)
    return value / divisor, value % divisor
end

local quotient: Int, remainder = divide(10, 3)
left, right = right, left
```

Tuple projection is one-based and uses an in-range literal index.

## Control Flow

Conditions are exactly `Boolean`; Pop has no Lua truthiness.

```luau
if ready then
    start()
elseif waiting then
    poll()
else
    stop()
end

local label = if ready then "ready" else "waiting"
```

Loops keep Luau block forms:

```luau
while running do
    tick()
end

repeat
    attempts += 1
until attempts == 3

for index = 1, count do
    visit(index)
end

for index = 10, 1, -1 do
    if shouldSkip(index) then
        continue
    end
    if shouldStop(index) then
        break
    end
end

for value in values do
    visit(value)
end
```

Numeric `for` ranges use one fixed integer type. Generalized `for value in
source` requires an exact nominal `Iterable<T>` or `Iterator<T>` implementation;
it performs no dynamic member lookup or implicit cleanup. Multiple bindings are
accepted only when each item is a fixed tuple of matching arity. Arrays,
insertion-ordered tables, `List<T>`, `Range<TInteger>`, and explicit nominal
iterators implement this traversal in the current compiler. Structural
length/key-set mutation is rejected when statically proven and otherwise traps
at the next iterator operation; replacing an existing element remains visible.

Compound assignments are `+=`, `-=`, `*=`, `/=`, `%=`, and `..=`. Their target
and right side evaluate once.

Exhaustive union/result/error matching uses `match`, `when`, and `then`:

```luau
match state
when LoadState.Idle then
    wait()
when LoadState.Loading(progress) then
    show(progress)
when LoadState.Ready(data) then
    consume(data)
end
```

Initial matches have no wildcard arms, guards, or expression-valued form. `_`
may ignore a case payload.

## Optional Values and Results

Use `T?` for absence. Narrow with a nil comparison or bind the present value
with `if local`/`while local`. Presence is distinct from truthiness.

```luau
if local player = findPlayer(id) then
    show(player)
else
    showMissing(id)
end

local port = configuredPort ?? DEFAULT_PORT
```

Postfix `?` propagates only optional absence from a function returning one
optional result:

```luau
private function parentName(player: Player?): String?
    local present = player?
    return findParentName(present)
end
```

Expected failures use `Result<T, TError>`. Prefix `try` propagates only the exact
same error type; conversion requires an exhaustive match.

```luau
public function loadName(path: String): Result<String, LoadError>
    local player = try loadPlayer(path)
    return Result.Ok(player.name)
end
```

Do not use postfix `?` for results, exceptions, implicit error conversion, or
unchecked unwraps.

Register deterministic lexical cleanup with `defer`. It runs in LIFO order for
normal exits, returns, result propagation, loop control, panic unwinding, and
cancellation; runtime traps do not promise cleanup.

```luau
local handle = try openHandle(path)
defer
    closeHandle(handle)
end
```

The initial cleanup body cannot return, break, continue, propagate, register
another defer, or suspend.

## Collections

Arrays are fixed-length, mutable, homogeneous, contiguous, and one-based.

```luau
local values = Array.create<<Int>>(count, 0)
local length = Array.length(values)
local maybe: Int? = values[index]
local value: Int = Array.get(values, index)
values[index] = value + 1
Array.fill(values, 0)
```

Ordinary array reads return `T?`; `Array.get` and invalid writes trap on bounds.
Arrays never grow implicitly.

Use `List<T>` for accepted one-based growable storage. `Sequence.map` and
`Sequence.filter` create lazy typed iterator adapters, `Sequence.fold` consumes
eagerly, and `Sequence.collect` materializes a list. These are ordinary Pop
implementations, not compiler-name dispatch.

`Range.create` constructs an inclusive immutable fixed-integer range value:

```luau
local ascending = Range.create(1, 5)
local descending = Range.create(5, 1, -2)

for value in descending do
    visit(value)
end
```

The arguments must resolve to one exact integer type. A zero step is rejected
statically when known and otherwise traps. Numeric `for` remains the shortest
form when the range does not need to be stored or passed.

Tables are invariant typed associative collections. Lookup returns an optional;
assignment inserts or replaces in deterministic insertion order.

```luau
local scores: {[String]: Int} = { alice = 10 }
local score: Int? = scores["alice"]
scores["bruno"] = 12
```

Assigning `nil` is not deletion. Do not use a table as a record, object, Module,
namespace, or dynamic property bag.

## Compile Time, Attributes, and Documentation

Attributes are typed PascalCase compile-time values placed before declarations.

```luau
@CompileTime
private function square(value: Int): Int
    return value * value
end

@Serializable(version = 2)
public record Player
    name: String
end
```

Compile time is deterministic and capability-limited: no ambient filesystem,
network, process, clock, random, environment, `eval`, or source injection.

Public API documentation uses `---` with checked XML and precedes attributes:

```luau
--- <summary>
--- Finds a player by identifier.
--- </summary>
---
--- <param name="id">
--- The player identifier.
--- </param>
---
--- <returns>
--- The player, or nil when absent.
--- </returns>
public function findPlayer(id: PlayerId): Player?
end
```

Do not treat documentation or attributes as runtime reflection.

Every non-empty XML element uses separate opening, body, and closing `---`
lines. Put exactly one empty `---` line between sibling top-level elements.
Do not write a non-empty `<summary>`, `<param>`, `<returns>`, or other element
inline. A genuinely empty element such as `--- <inheritdoc/>` may self-close.

## Formatting

- Use four spaces, never emitted tabs.
- Omit semicolons.
- Put one statement on a line.
- Put one item per line with a trailing comma in multiline argument/data lists.
- Put spaces around binary operators, not unary operators.
- Target 100 columns.
- Use `elseif`, not `else if`.
- Use braces only for data/initializers, never executable blocks.
- Let the canonical formatter own whitespace once `pop format` is available;
  until then, follow nearby canonical source.
