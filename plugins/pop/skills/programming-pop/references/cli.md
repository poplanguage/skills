# Pop CLI and Packages

Use this reference for `pop`, `bubble.toml`, source layout, and validation. The
CLI is under active construction; always make `pop --help` the local authority
for executable commands.

## Ownership Model

```text
Workspace
└── Package
    └── Bubble
        └── Module
            └── Item
```

- An Item is a declaration/member/case.
- A Module is one `.pop` file and the `private` boundary.
- A Bubble is an independently compiled target and the `internal` boundary.
- A Package is a versioned directory with `[package]` in `bubble.toml`.
- A Workspace shares resolution, lock, cache, and policy without widening
  visibility.
- A namespace organizes names and is not one of these ownership units.

## Conventional Package

```text
calculator/
├── bubble.toml
├── src/
│   ├── lib.pop
│   ├── main.pop
│   └── bin/
│       └── migrate.pop
├── tests/
├── examples/
├── benchmarks/
└── resources/
```

Minimal manifest:

```toml
[package]
name = "Studio.Calculator"
version = "0.1.0"
edition = "2026"

[dependencies]
StudioData = "2.1"
```

Manifest keys use `camelCase`; Package/Bubble identities use PascalCase
components. Paths guide discovery but do not define semantic identity.

Conventional roots:

- `src/lib.pop`: default library Bubble;
- `src/main.pop`: default binary Bubble;
- `src/bin/*.pop`: additional single-Module binaries;
- `src/bin/<name>/main.pop`: multi-Module binary rooted in that directory;
- `tests/*.pop`, `examples/*.pop`, `benchmarks/*.pop`: separate Bubbles that use
  only the library's public surface plus declared dependencies.

A minimal binary entry is:

```luau
namespace Calculator

function main()
    print(42)
end
```

The full accepted entry is `private function main(arguments: Array<String>): Int`.
Parameters may be absent or exactly that array; the result may be absent or
exactly `Int`. Program arguments exclude the executable path.

## Implemented Bootstrap Commands

At the time this skill was authored, the compiler executable implements:

```text
pop check <source.pop> [--dump <hir|mir|ll>]...
pop check --manifestPath <bubble.toml>
pop build <source.pop> --output <executable>
pop build --manifestPath <bubble.toml>
pop documentation --manifestPath <bubble.toml>
pop transpile <source.pop> --to c
pop run <source.pop> [-- <arguments>...]
pop run --manifestPath <bubble.toml> [-- <arguments>...]
```

Examples:

```bash
pop check src/main.pop
pop check src/main.pop --dump hir --dump mir
pop build src/main.pop --output target/app
pop check --manifestPath bubble.toml
pop build --manifestPath bubble.toml
pop documentation --manifestPath bubble.toml
pop run src/main.pop -- first "two words"
pop run --manifestPath bubble.toml -- first
```

`check` on a direct source path creates an ephemeral compilation context. It is
for compiler inspection and does not resolve a real Package or dependencies.
HIR/MIR/LLVM dumps are deterministic debug text for that compiler version, not
stable interchange formats.

The direct `build` and `run` paths target LLVM/native execution. `transpile --to
c` is experimental, deliberately restricted, and not a parity target. Do not
change ordinary Pop code merely to satisfy the C experiment.

Manifest-driven `check`, `build`, and `run` discover conventional Bubbles and
virtual Workspace default members. They resolve exact local-path Package
dependencies through public metadata, generate one canonical `bubble.lock`, and
share the Workspace `target/` root. `documentation` emits checked public XML for
selected library Bubbles. Use `--locked`, `--offline`, or `--frozen` after the
installed command confirms those options.

Registry and exact-Git requirements are parsed, but current bootstrap resolution
requires an already resolved lock for those transports. Do not describe network
fetching, publishing, or a registry client as implemented.

## Accepted but Not Necessarily Implemented

The architecture reserves the following complete command family:

```text
pop new                 pop initialize
pop check               pop build
pop transpile           pop run
pop test                pop benchmark
pop documentation       pop format
pop lint                pop fix
pop add                 pop remove
pop update              pop tree
pop metadata            pop package
pop publish             pop install
pop clean
```

Do not run or recommend a reserved command until `pop --help` confirms it in the
installed toolchain. In particular, do not assume the full formatter, linter,
registry transport, test runner, publishing flow, or LSP is available.

The planned long selectors use complete camelCase spellings such as
`--manifestPath`, `--platformTarget`, and `--messageFormat`. Avoid inventing
shortened primary flags.

## Validation Strategy

1. Run `pop --help` and record the current command surface.
2. For syntax/type work, run `pop check` on the affected standalone Module when
   possible.
3. For executable behavior, use `pop run` or build and execute the output.
4. For real Package work, prefer manifest-driven commands and inspect the
   generated lock/artifact behavior rather than assuming registry support.
5. Run project-specific tests from its repository instructions.
6. Never report a reserved roadmap command as successfully validated unless it
   was actually executed.

`Pop.Standard` is automatically referenced. Official extensions are ordinary,
independently selected Packages, not implicit standard-library dependencies.
Avoid guessing their APIs or availability.
