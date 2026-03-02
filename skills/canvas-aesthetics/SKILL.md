---
name: canvas-aesthetics
description: >-
  Layout guidelines for achieving aesthetics and readability when creating or editing .canvas files.
  Use alongside json-canvas when creating or reviewing .canvas files.
---

# canvas-aesthetics

Guidelines for the aesthetic quality and readability of the **overall canvas composition**.
If json-canvas answers "how to write it", this skill answers "how to lay it out beautifully".

## 1. Canvas Proportions

- **Aspect ratio**: 1:1 ~ 3:1 (width ≥ height)
- **Maximum height**: 2000px (≤20 nodes), absolute upper limit 3000px
- **Zoom-to-fit readability formula**: `min(1400/width, 900/height) ≥ 0.5`
  - Values below 0.5 mean text becomes difficult to read at zoom-to-fit

## 2. Layout Patterns

| Pattern | Best for | Aspect ratio |
|---------|----------|-------------|
| Horizontal flow | Sequential processes, pipelines | 2:1 ~ 3:1 |
| Swimlane / grid | Parallel workflows | 1:1 ~ 1.5:1 |
| Hub-and-spoke | Central concept + related items | ~1:1 |
| Vertical flow | Short sequences (≤8 nodes) **only** | 0.7:1 ~ 1:1 |

- Vertical chain exceeding 8 nodes → switch to multi-column flow or swimlane
- Repeating patterns (A→B ×N) → compress to 1 cycle + annotation

## 3. Visual Hierarchy — 3 Levels

| Level | Purpose | Heading | Width | Color |
|-------|---------|---------|-------|-------|
| Landmark | Phase titles, key decisions | `##` H2 | 400-600 | Strong preset |
| Standard | General steps | `###` H3 | 280-400 | Colorless / light |
| Annotation | Legends, metadata | Body only | 200-300 | Colorless |

- Use a minimum of 2 visual hierarchy levels

## 4. Color Strategy

- Color only flow-critical nodes (commands, decisions, transitions)
- General steps: colorless; groups: different color from inner nodes
- Edges: color only exception paths
- **Maximum 4 colors per canvas**

## 5. Information Density

- Text nodes ≤ 25 per canvas (split into file nodes if exceeded)
- Flow nodes ≤ 4 visual lines
- Groups ≤ 4 per canvas

## 6. Auxiliary Content Placement

- Legends and metadata: inside the main bounding box, within 200px of the nearest flow element
- Large auxiliary content (300px+): separate into file nodes

## 7. Pre-Flight Checklist

Verify the following items after creating or editing a canvas:

1. Zoom-to-fit readability: `min(1400/W, 900/H) ≥ 0.5`
2. Aspect ratio: width ≥ height
3. Space utilization ≥ 30%
4. Single column height ≤ 2000px
5. Flow node colors ≤ 4 types
6. Text nodes ≤ 25
7. Visual hierarchy ≥ 2 levels
