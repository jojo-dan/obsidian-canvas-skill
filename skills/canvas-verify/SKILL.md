---
name: canvas-verify
description: >-
  Validates the JSON integrity and canvas-aesthetics Pre-Flight Checklist (7 items) of a .canvas file.
  Can be run standalone or called from within the canvas skill pipeline. Also performs visual validation via Obsidian screenshot.
  Use when: /canvas-verify "file-path", verify canvas, inspect canvas
argument-hint: '"file-path.canvas"'
---

# canvas-verify

A skill that validates the integrity of `.canvas` files. Executes in order: JSON parsing → Pre-Flight Checklist → visual validation.

## Usage

```
/canvas-verify "file-path.canvas"
```

Supports both standalone execution and internal calls from the `canvas` skill pipeline.

## Validation Procedure

### Step 1: JSON Parsing Validation

Read the file and attempt JSON parsing.

**Parsing succeeds:** proceed to structural validation

**Parsing fails:** report error location and content, then stop

```
[ERROR] JSON parsing error
  Location: line 42, column 15
  Content: Unexpected token '}' after value
  Context: "...\"color\": \"1\"}"
```

### Step 2: Node/Edge Integrity Validation

Inspect structure against JSON Canvas spec.

Inspection items:
- Presence of `nodes` array
- Presence of `edges` array (empty array allowed if absent)
- Required fields for each node: `id`, `type`, `x`, `y`, `width`, `height`
- Required fields for each edge: `id`, `fromNode`, `toNode`
- Verify that `fromNode`/`toNode` in edges reference actual node ids (detect dangling edges)

On error:
```
[ERROR] Node integrity error
  Node id: "node-3"
  Missing field: height
```

```
[ERROR] Dangling edge
  Edge id: "edge-2"
  Referenced node id "node-99" does not exist
```

### Step 3: Pre-Flight Checklist Validation

Inspect the 7 items from the canvas-aesthetics guidelines.

Calculations:
- **Total canvas W/H**: calculated from max of (x+width) and (y+height) across all nodes
- **Number of color types**: unique values of `color` field in nodes and edges
- **Text node count**: number of nodes with `type: "text"`
- **Visual hierarchy**: determined by mixing of heading levels (`##`, `###`)
- **Aspect ratio**: W ÷ H
- **Space utilization**: sum of all node areas ÷ (W × H) × 100

**Checklist results table:**

| # | Item | Threshold | Measured | Result |
|---|------|-----------|----------|--------|
| 1 | Zoom-to-fit readability | `min(1400/W, 900/H) ≥ 0.5` | 0.61 | PASS |
| 2 | Aspect ratio | width ≥ height | 1.8:1 | PASS |
| 3 | Space utilization | ≥ 30% | 42% | PASS |
| 4 | Single column height | ≤ 2000px | 1400px | PASS |
| 5 | Flow node color count | ≤ 4 types | 3 types | PASS |
| 6 | Text node count | ≤ 25 | 8 | PASS |
| 7 | Visual hierarchy levels | ≥ 2 levels | 2 levels | PASS |

Each item is marked as `PASS` or `FAIL`. FAIL items include measured value and threshold in output.

### Step 4: Visual Validation

Capture the canvas using the `obsidian-cli` skill's screenshot feature and have the LLM evaluate it multimodally.

```
obsidian-cli screenshot --file "file-path"
```

Evaluation items:
- No node overlap
- No text clipping
- Overall layout readability

**If Obsidian is not running:**
- Skip visual validation
- Output `[WARN] Skipping visual validation because Obsidian is not running.`

## Final Output

### All PASS

```
## canvas-verify result

File: ./example.canvas
JSON parsing: OK
Nodes: 8 / Edges: 6

| # | Item | Threshold | Measured | Result |
|---|------|-----------|----------|--------|
| 1 | Zoom-to-fit readability | ≥ 0.5 | 0.61 | PASS |
| 2 | Aspect ratio | width ≥ height | 1.8:1 | PASS |
| 3 | Space utilization | ≥ 30% | 42% | PASS |
| 4 | Single column height | ≤ 2000px | 1400px | PASS |
| 5 | Flow node color count | ≤ 4 types | 3 types | PASS |
| 6 | Text node count | ≤ 25 | 8 | PASS |
| 7 | Visual hierarchy levels | ≥ 2 levels | 2 levels | PASS |

Visual validation: PASS

Total result: PASS (7/7)
```

### FAIL items exist

```
## canvas-verify result

File: ./example.canvas
JSON parsing: OK
Nodes: 8 / Edges: 6

| # | Item | Threshold | Measured | Result |
|---|------|-----------|----------|--------|
| 1 | Zoom-to-fit readability | ≥ 0.5 | 0.42 | FAIL |
| 2 | Aspect ratio | width ≥ height | 0.8:1 | FAIL |
| 3 | Space utilization | ≥ 30% | 38% | PASS |
| 4 | Single column height | ≤ 2000px | 1800px | PASS |
| 5 | Flow node color count | ≤ 4 types | 3 types | PASS |
| 6 | Text node count | ≤ 25 | 8 | PASS |
| 7 | Visual hierarchy levels | ≥ 2 levels | 1 level | FAIL |

Visual validation: [WARN] Skipping visual validation because Obsidian is not running.

Total result: FAIL (4/7)
Failed items: 1, 2, 7
```

### JSON parsing error

```
## canvas-verify result

File: ./example.canvas
JSON parsing: ERROR

[ERROR] JSON parsing error
  Location: line 42, column 15
  Content: Unexpected token '}' after value
  Context: "...\"color\": \"1\"}"

Pre-Flight Checklist: skipped (JSON parsing failed)
Visual validation: skipped (JSON parsing failed)

Total result: FAIL (JSON error)
```

## Return Value

Structure returned when called from the canvas skill pipeline:

```
{
  "status": "PASS" | "FAIL" | "ERROR",
  "pass_count": 7,
  "total": 7,
  "failed_items": [],       // list of FAIL item numbers
  "json_error": null,       // JSON parsing error message (if any)
  "visual_skipped": false   // whether visual validation was skipped
}
```
