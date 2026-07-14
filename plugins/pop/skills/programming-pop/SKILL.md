---
name: programming-pop
description: Write, edit, review, explain, run, and troubleshoot Pop Lang source, `.pop` modules, `bubble.toml` packages, `pop` CLI commands, and `popup` toolchain setup. Use whenever a task involves Pop Lang syntax, types, control flow, declarations, idiomatic API shape, package layout, compiler commands, or toolchain installation. Distinguish implemented behavior from roadmap contracts and avoid inventing standard-library APIs.
---

# Program Pop Lang

Write native, strongly and statically typed Pop Lang that keeps its Luau-shaped
syntax. Prefer accepted language contracts over guesses from another language.

## Start Every Task

1. Inspect the repository for `AGENTS.md`, `.pop` files, `bubble.toml`, and local
   documentation. Repository rules and accepted ADRs override this skill.
2. Check the installed tool with `pop --help` before relying on a command. The
   implementation is evolving faster than the planned CLI surface.
3. Load only the reference needed for the task:
   - [syntax.md](references/syntax.md) for grammar and language constructs;
   - [idioms.md](references/idioms.md) for design, naming, and common mistakes;
   - [cli.md](references/cli.md) for packages and `pop` commands;
   - [popup.md](references/popup.md) for installing/selecting toolchains.
4. Search nearby Pop source for established project patterns before adding new
   structure or names.
5. Run the narrowest supported check, then build or run when the task requires
   executable validation. Report unsupported or unrun validation honestly.

## Keep These Invariants

- Write `Pop Lang` in prose, `.pop` for source, and `pop` for the language CLI.
- Use `namespace`, `using`, `function`, `local`, and `end`; do not import braces,
  semicolons, `export`, or import syntax from JavaScript, Rust, C#, or D.
- Keep every runtime value and operation statically typed. Never invent `Any`,
  `Dynamic`, unchecked calls, implicit globals, or runtime string lookup.
- Put one file-scoped namespace in each Module. `using` changes name lookup only.
- Namespace declarations default to `internal`. Binary-root `function main` is
  the sole default-`private` exception. Interface members remain implicitly
  public.
- Annotate every parameter. Omitting a function result means no returned values;
  it never requests return-type inference.
- Prefer functions, records, unions, arrays/tables, and explicit capabilities.
  Use classes only for identity, encapsulated mutable lifecycle, or real runtime
  dispatch.
- Use `Result<T, E>` for expected failure, `T?` for absence, and `defer` for
  deterministic cleanup. Do not turn ordinary failure into panic or exceptions.
- Put every non-empty XML documentation element on separate opening, content,
  and closing `---` lines. Separate sibling contract elements with one empty
  `---` line; retain self-closing syntax only for genuinely empty elements.
- Preserve the Item -> Module -> Bubble -> Package -> Workspace vocabulary.
- Do not invent public-library functions. Inspect declarations or keep examples
  at the language/CLI level when the library contract is unavailable.

## Canonical Shape

```luau
namespace Example.Calculator

--- <summary>
--- Describes division failures.
--- </summary>
public error DivideError
    --- <summary>
    --- The divisor is zero.
    --- </summary>
    Zero
end

--- <summary>
--- Divides one integer by another.
--- </summary>
---
--- <param name="value">
--- The dividend.
--- </param>
---
--- <param name="divisor">
--- The divisor.
--- </param>
---
--- <returns>
--- The integer quotient.
--- </returns>
---
--- <error type="DivideError.Zero">
--- The divisor is zero.
--- </error>
public function divide(value: Int, divisor: Int): Result<Int, DivideError>
    if divisor == 0 then
        return Result.Error(DivideError.Zero)
    end

    return Result.Ok(value / divisor)
end
```

For a binary root:

```luau
namespace Example.App

function main()
    print("Hello from Pop")
end
```

Do not add `public` to the binary `main`; its accepted omitted visibility is
private. A library function named `main` is ordinary and defaults to internal.

## Work With an Evolving Toolchain

Treat three kinds of information separately:

| Status | How to use it |
| --- | --- |
| Implemented | Confirm through nearby tests/source or `pop --help`, then use it. |
| Accepted architecture | Use for new design, but verify the current compiler supports it before promising execution. |
| Roadmap/planned | Mention as future work only; do not generate code that depends on it. |

When compiler behavior and accepted architecture disagree, do not work around it
with dynamic behavior or foreign syntax. Report the gap and follow the repository's
architecture-to-tests-to-implementation process.

## Validate

For a standalone Module, prefer the implemented bootstrap commands:

```bash
pop check path/to/source.pop
pop run path/to/source.pop
```

Use package commands only after confirming support with `pop --help`. Local-path
Package resolution and manifest-driven check/build/run/documentation are present
in the current repository, but registry transport, LSP, the complete formatter,
and the full standard library remain unfinished.
