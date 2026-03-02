# obsidian-canvas-skill

Claude Code skills for generating and validating Obsidian `.canvas` files from natural language.

This collection complements the [`json-canvas`](https://github.com/obsidianmd/json-canvas) skill by adding a full pipeline: layout planning, file generation, automated validation, visual verification, and correction loops.

## Skills Included

| Skill | Description |
|-------|-------------|
| `canvas` | 5-step pipeline: layout planning → json-canvas generation → validation → visual check → correction loop |
| `canvas-verify` | Validates JSON integrity and 7-item Pre-Flight Checklist for any `.canvas` file |
| `canvas-aesthetics` | Layout guidelines for aesthetics and readability (used as a reference by the other two skills) |

## Prerequisites

- [Claude Code](https://claude.ai/code) — required to run skills
- [Obsidian](https://obsidian.md) — required to open and view `.canvas` files
- [`json-canvas` skill](https://github.com/obsidianmd/json-canvas) — used by `canvas` in Step 2 to write the file
- [`obsidian-cli`](https://github.com/open-source-labs/obsidian-cli) skill — used for screenshot-based visual validation (optional; validation falls back to JSON-only if Obsidian is not running)

## Installation

### Option 1: Plugin install (recommended)

```
/plugin add https://github.com/jojo-dan/obsidian-canvas-skill
```

### Option 2: Global skills directory

```bash
git clone https://github.com/jojo-dan/obsidian-canvas-skill ~/.claude/skills/obsidian-canvas-skill
```

### Option 3: Project-local skills

```bash
git clone https://github.com/jojo-dan/obsidian-canvas-skill .claude/skills/obsidian-canvas-skill
```

After installation, the skills are available as `/canvas`, `/canvas-verify`, and `/canvas-aesthetics` in your Claude Code session.

## Usage

### Generate a canvas

```
/canvas "5-step CI/CD pipeline diagram"
```

Output is saved as `{title}.canvas` in the current working directory by default.

```
/canvas "team onboarding flow" "docs/onboarding.canvas"
```

Specify an explicit output path as the second argument.

### Validate an existing canvas

```
/canvas-verify "path/to/file.canvas"
```

Runs 7-item Pre-Flight Checklist and optional visual validation via Obsidian screenshot.

### Reference aesthetics guidelines

```
/canvas-aesthetics
```

Loads the layout and aesthetics guidelines for use when manually designing a canvas.

## How It Works

The `canvas` skill runs a 5-step pipeline:

1. **Layout Planning** — LLM designs node structure, layout pattern, and coordinates
2. **File Generation** — `json-canvas` skill writes the `.canvas` file
3. **Validation** — `canvas-verify` checks JSON integrity and 7 aesthetic criteria
4. **Visual Check** — `obsidian-cli` takes a screenshot; LLM evaluates the result
5. **Correction Loop** — Up to 3 automated correction iterations if any check fails

If Obsidian is not running, Steps 4 and 5 fall back to JSON-only validation.

## Relationship with json-canvas

`json-canvas` handles the low-level serialization — writing valid JSON Canvas format files. This skill collection sits on top of it and adds:

- Natural language input processing
- Aesthetic layout planning
- Automated quality validation
- Visual verification loop

Think of `json-canvas` as the engine and `obsidian-canvas-skill` as the driver.

## Demo

Open `examples/demo.canvas` in Obsidian to see a rendered example of the canvas pipeline diagram.

## License

MIT
