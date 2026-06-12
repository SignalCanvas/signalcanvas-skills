# SignalCanvas Claude Code Skills

Claude Code skills for SignalCanvas development. Install these so Claude follows consistent conventions across all contributors.

## Install

```bash
git clone https://github.com/ByteBard97/signalcanvas-skills ~/.claude/skills/signalcanvas
```

Claude Code picks up skills from `~/.claude/skills/` automatically.

## Skills

| Skill | When to use |
|-------|-------------|
| `vue-best-practices` | Any `.vue` file, composable, or Pinia store in any Vue 3 project |
| `signalcanvas-patchlang` | Writing, editing, or validating `.patch` files (language syntax reference) |
| `patchlang-architecture` | Architecture boundary between PatchLang (`.patch`) and the JSON sidecar (`.layout.json`) — use when touching device data, connections, the canvas scene store, the emitter, the loader, or the sidecar JSON |
| `code-rules` | Any code task — enforces file size, DRY, naming, and error handling rules |
| `signalcanvas-builder` | Building or importing a signal flow into SignalCanvas — from conversation, CSV/XLSX patch lists, or screenshots of Visio/handwritten diagrams |
| `stock-library-builder` | Building, editing, or auditing device templates in the SignalCanvas stock library (`src/data/stdlib/`) |
| `context-handoff` | Write a HANDOFF.md so you can clear context and resume cleanly in a fresh session |

## Slash Commands

Run any of these from the FrontendV1 repo root.

| Command | Plugin | What it does |
|---------|--------|--------------|
| `/scbt` | `signalcanvas-testing` | Run the browser trace integration tests (Playwright) |
| `/scft` | `signalcanvas-testing` | Run the label scanner on a specific `.patch` file |
| `/scpt` | `signalcanvas-testing` | Run the signal flow protocol test suite |
| `/scpkg` | `signalcanvas-packaging` | Cut a new beta release — bumps version, updates changelog/BETA-README, builds the macOS `.pkg` installer |

## Usage

Invoke a skill explicitly for best results:

```
Use vue-best-practices skill. Add a settings panel to CanvasToolbar.
```

Or Claude will pick them up automatically based on the task context.

## Contributing

Edit the `SKILL.md` in any skill directory. The `vue-best-practices` skill is based on [vuejs-ai/skills](https://github.com/vuejs-ai/skills).
