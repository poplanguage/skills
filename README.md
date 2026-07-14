# Pop Lang Agent Skills

Curated skills that help coding agents write idiomatic, strongly and statically
typed Pop Lang without importing syntax or conventions from other languages.

## Included

| Plugin | Skill | Purpose |
| --- | --- | --- |
| [`pop`](plugins/pop/) | `programming-pop` | Pop Lang syntax, idioms, packages, the `pop` CLI, and Popup toolchain management |

The skill keeps its prompt small and loads detailed references only when needed.
It distinguishes implemented tool behavior from accepted architecture and
roadmap work.

## Install

### Codex

Register this repository as a plugin marketplace, then install the `pop` plugin:

```bash
codex plugin marketplace add poplanguage/skills
```

Open `/plugins` in Codex, select the Pop Lang marketplace, and install `pop`.
The individual skill can also be installed directly from:

```text
https://github.com/poplanguage/skills/tree/master/plugins/pop/skills/programming-pop
```

### Copilot CLI or Claude Code

```text
/plugin marketplace add poplanguage/skills
/plugin install pop@pop-lang-skills
```

Restart the client after installation, then inspect `/skills`.

### Local development

Point a compatible Agent Skills client at
`plugins/pop/skills/programming-pop`, or copy that directory into the client's
local skills directory.

## Scope

This repository teaches agents how to program in Pop Lang. It intentionally does
not promise unfinished standard-library APIs, Package registry behavior, LSP
integration, or roadmap-only commands. Update the references alongside accepted
Pop Lang architecture and released tool behavior.

## License

MIT. See [LICENSE](LICENSE).
