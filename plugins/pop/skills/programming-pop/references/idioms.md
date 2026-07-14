# Idiomatic Pop Lang

Use this reference for design choices and reviews. Pop Lang is procedural,
functional, data-oriented, and optionally object-oriented. It is not “everything
is an object” and it is not dynamically typed Lua.

## Preferred Abstractions

Choose the first form that expresses the real semantics:

1. local values and plain functions;
2. records and tagged unions;
3. typed arrays/tables and generic algorithms;
4. Modules and namespaces;
5. explicit functions, data, and capabilities;
6. small nominal interfaces for true polymorphic boundaries;
7. classes for identity, encapsulated mutable lifecycle, or runtime dispatch;
8. inheritance only for intentional long-term substitutability.

Namespace functions replace static utility classes and singleton module objects.

```luau
namespace Image

public function resize(image: ImageData, width: Int, height: Int): ImageData
end
```

Do not introduce `ImageService`, `ImageManager`, `ImageFactory`, a returned module
table, or a universal object solely to organize this operation.

## Data First

Use records for configuration, messages, requests, AST nodes, coordinates, and
other product data. Keep alternatives explicit with unions.

```luau
public record Position
    x: Float
    y: Float
end

public function move(position: Position, dx: Float, dy: Float): Position
    return position with {
        x = position.x + dx,
        y = position.y + dy,
    }
end
```

Use a class when the value owns a live connection, resource, actor identity, or
mutable invariant that must be encapsulated. A class is not a DTO spelling.

## Make Dependencies Explicit

Pass a capability, function, record of operations, or small interface. Do not
resolve dependencies from global mutable state, runtime type names, or service
locators.

```luau
internal record UserOps
    load: function(id: UserId): Result<User, UserError>
    save: function(user: User): Result<(), UserError>
end
```

Opaque handles are appropriate for real native resources or protocol state, but
their lifecycle and cleanup must remain direct and explicit.

## Model Failure Precisely

- Use `T?` only when absence is the whole outcome.
- Use a nominal `error` plus `Result<T, E>` when callers must distinguish failures.
- Use `try` only when the caller returns the exact same `E`.
- Convert error families with an exhaustive `match`, preserving information.
- Reserve panic for broken invariants. Runtime traps are closed failures such as
  bounds or checked numeric conversion violations.
- Register resource cleanup immediately after acquisition with `defer`.

Do not invent exception hierarchies, implicit conversions, message-only errors,
or finalizers required for correctness.

## Preserve Static Meaning

Never solve uncertainty with:

- `Any`, `Dynamic`, an untyped table, or heterogeneous untyped collections;
- string-based member, type, function, dependency, or attribute lookup;
- implicit globals or fallback opcodes;
- source generation from strings or unrestricted runtime reflection;
- Lua metatables as the implementation model for records, classes, Modules, or
  namespaces;
- backend-specific behavior leaking into source, HIR, or MIR.

Use explicit unions, interfaces, optional/result types, typed adapters, and
checked FFI boundaries instead.

## Keep Names Native and Complete

Use complete words, then rely on namespace context to keep calls short:

```luau
Json.encode(value)
File.open(path)
Sequence.map(values, transform)
```

Avoid repeated context such as `Json.encodeJsonValue`, and avoid arbitrary
truncations such as `Iter`, `Config`, `Mgr`, or `Util`. Established forms such as
`Json`, `Http`, `Io`, `Utf8`, `Ffi`, `Gc`, `Guid`, `Async`, `Ui`, and `Ai` are
accepted exceptions.

Do not add suffixes such as `Builder`, `Factory`, `Manager`, `Provider`, `Service`,
`Utility`, or `Helper` when a record and free function express the operation.

## Keep Costs Legible

- Prefer views or explicit buffers only when their ownership/lifetime contract is
  accepted and visible.
- Do not hide copying, allocation, I/O, blocking, suspension, or native crossing
  behind a convenience abstraction.
- Arrays are fixed-length; do not simulate silent growth.
- Table lookup is optional and table insertion may allocate.
- String concatenation and primitive formatting may allocate.
- Generic runtime code is statically specialized in the current accepted model;
  do not introduce runtime dictionaries or type inspection.

When documenting a public API, state allocation, ownership/copy behavior,
complexity, blocking/suspension, thread safety, and relevant failure conditions.

## Common Agent Corrections

| Do not write | Write instead |
| --- | --- |
| `export function run()` | `public function run()` |
| `function run(value)` | `function run(value: Value)` |
| omitted return type while returning a value | declare the exact result type |
| `if value then` for an optional | `if local present = value then` |
| postfix `?` for `Result` | prefix `try` |
| `try/catch` exceptions | `Result`, `try`, and exhaustive `match` |
| `value.toString()` | `String(value)` for the closed primitive set |
| `${value}` | backtick interpolation: `{value}` |
| zero-based array assumptions | one-based indexing |
| array append/growth | use the accepted growable collection when available |
| `else if` | `elseif` |
| `{ ... }` statement blocks | `do`/`then` plus `end` |
| `import`/`require` | `using` plus a declared Package dependency |
| a table as an object/namespace | a record, class, Module, or namespace |
| `Option<T>` by guess | `T?`, unless the project declares otherwise |

## Review Checklist

- Does every Module have exactly one file-scoped namespace?
- Are parameters and returned values explicitly typed?
- Is omitted visibility intentionally internal or the binary-main exception?
- Are conditions Boolean rather than truthy?
- Are optional and result propagation distinct?
- Does every non-empty XML documentation element use the canonical multiline
  shape, with empty `---` separators and no trailing whitespace?
- Are union/error matches exhaustive?
- Is cleanup deterministic and registered after acquisition?
- Are records, classes, tables, namespaces, Modules, and Packages kept distinct?
- Are names canonical and calls concise without truncation?
- Did the change avoid relying on planned library or CLI behavior?
- Was validation actually run on the supported interpreter/LLVM path?
