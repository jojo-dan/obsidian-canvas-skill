---
name: canvas
description: >-
  Generate an Obsidian .canvas file from natural language requirements using a 5-step pipeline.
  Layout planning → json-canvas generation → canvas-verify validation → visual check → correction loop.
  Use when: /canvas "requirements", create a canvas, generate canvas
argument-hint: '"requirements description" [output path]'
---

# canvas

A pipeline skill that takes natural language requirements and generates, validates, and corrects an Obsidian `.canvas` file.

## Usage

```
/canvas "requirements"
/canvas "requirements" "output-path/filename.canvas"
```

## Output Path Rules

1. If a path is specified in the argument: use that path
2. If no path is specified: save as `{title}.canvas` in the current working directory (CWD)
3. If the desired output location is ambiguous: ask the user to confirm the path before proceeding

## Pipeline (5 Steps)

### Step 1: Layout Planning

The LLM analyzes requirements and designs the canvas structure.

Design items:
- Node list: each node's id, text, role (landmark/standard/annotation)
- Node placement: x/y coordinates, width/height
- Edge connections: fromNode → toNode, labels
- Overall canvas bounding box size (width × height)
- Layout pattern selection (horizontal flow / swimlane / hub-and-spoke / vertical flow)

Apply canvas-aesthetics guidelines:
- Aspect ratio 1:1 ~ 3:1 (width ≥ height)
- Zoom-to-fit formula: `min(1400/W, 900/H) ≥ 0.5`
- Visual hierarchy 2+ levels (Landmark/Standard/Annotation)
- Maximum 4 colors

### Step 2: json-canvas File Generation

Use the `json-canvas` skill to generate a `.canvas` file from the Step 1 plan.

- Determine file path according to output path rules
- Use json-canvas skill to create a valid JSON Canvas format file
- Record the generated file path

### Step 3: canvas-verify Validation

Call the `canvas-verify` skill to validate the generated file.

```
/canvas-verify "file-path"
```

- Check pass/fail for 7 Pre-Flight Checklist items
- Check for JSON parsing errors
- Receive validation results

Validation result:
- **All PASS**: proceed to Step 4 (visual check)
- **Any FAIL**: enter Step 5 (correction loop)

### Step 4: Visual Check

Capture the canvas using the `obsidian-cli` skill's screenshot feature and have the LLM evaluate it multimodally.

```
obsidian-cli screenshot --file "file-path"
```

Evaluation criteria:
- No node overlap
- No text clipping
- Overall layout matches intended pattern
- Adequate readability

**If Obsidian is not running:**
- Perform JSON integrity validation only
- Skip visual validation
- Output warning: `[WARN] Skipping visual validation because Obsidian is not running.`

Visual check result:
- **No issues**: complete, output file path
- **Issues found**: enter Step 5 (correction loop)

### Step 5: Correction Loop (max 3 iterations)

Automatically attempt corrections when validation or visual check finds failures.

**Correction procedure (per iteration):**

1. Review list of failed items
2. LLM formulates a correction plan for those items
3. Regenerate file with json-canvas skill (incorporating correction plan)
4. Re-run canvas-verify
5. Re-run visual check (if Obsidian is running)

**Correction limit:**
- Maximum 3 iterations
- If FAIL items remain after 3 iterations, stop the loop
- Output remaining failed items along with the file path and request manual review from user

**Correction loop status output example:**
```
[Correction 1/3] Failed items: Zoom-to-fit readability, aspect ratio
  → Correction plan: width 1800 → 1200, height unchanged
  → Regeneration complete
[Correction 2/3] Failed items: none → validation passed
```

## Final Output

```
## canvas complete

File: ./example.canvas
Validation: PASS (7/7)
Corrections: 1

Nodes generated: 8
Edges generated: 6
```

If failed items remain:
```
## canvas complete (with warnings)

File: ./example.canvas
Validation: PARTIAL (5/7)
Corrections: 3 (limit reached)

Unresolved items:
- [ ] Zoom-to-fit readability: 0.42 (threshold: ≥ 0.5)
- [ ] Aspect ratio: height > width

Manual review required.
```
