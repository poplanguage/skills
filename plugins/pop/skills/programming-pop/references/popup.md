# Popup Toolchain Manager

`popup` manages Pop Lang toolchains. `pop` compiles code and manages Pop
Packages; the two commands have deliberately separate responsibilities.

## Current Repository Implementation

The current standalone Popup repository is written in Crystal and requires
Crystal 1.20.1 or later to build:

```bash
shards install
shards build
crystal spec
```

Its implemented user command is:

```text
popup install [version]
```

Examples:

```bash
popup install
popup install v0.1.0
```

With no version, it queries the latest release. The current implementation:

- supports Linux `x86_64` and `aarch64` target selection;
- queries releases from `poplanguage/pop` on GitHub;
- selects the matching non-`.sha256` asset;
- downloads that asset into the current directory.

The documented bootstrap script is work in progress. Current code does not yet
provide the complete verified multi-toolchain state model below. Inspect
`popup --help` and the Popup repository before giving operational instructions.

## Accepted Target Contract

The accepted architecture expands Popup to:

```text
popup list [--installed|--available] [--includePrerelease]
popup install <version|stable>
popup default <version|stable>
popup run --toolchain <version|stable> -- <command> [arguments...]
popup update [<version|stable>]
popup uninstall <version>
popup doctor
popup self update
```

These commands are architectural targets, not evidence of current
implementation. Do not present them as usable without checking the installed
binary.

The target design uses immutable, verified, relocatable toolchains containing
the compiler, runtime, `Pop.Internal`, `Pop.Standard`, adapters, licenses, and a
signed file inventory. `popup` does not edit `bubble.toml`, resolve Packages, or
publish code.

Planned selection precedence is:

1. `popup run --toolchain ...`;
2. `POPUP_TOOLCHAIN`;
3. nearest ancestor `pop-toolchain.toml`;
4. the global default.

Selection must not download implicitly. Exact versions are preferred for
automation and checked-in pins; `stable` is the only initial moving channel.

## Safety Rules

- Never confuse `pop install` (a Package binary) with `popup install` (a whole
  Pop Lang toolchain).
- Do not bypass signatures, digests, expiry, rollback protection, or target
  checks once the production manager implements them.
- Do not request `sudo`, edit shell startup files silently, or overwrite a
  running toolchain in place.
- Do not treat a Git tag, asset filename, successful TLS request, or adjacent
  checksum from the same unsigned response as the final trust identity.
- Do not claim the present prototype already satisfies the production security
  architecture.
- Prefer exact version output and verified release metadata when diagnosing
  toolchain mismatches.

For current installation troubleshooting, inspect the Popup repository and its
tests rather than extrapolating from Rustup or another ecosystem's manager.
