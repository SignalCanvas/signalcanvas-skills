---
name: canvas-audit
description: >
  Use when verifying the VueFlow canvas after layout, routing, or merge changes —
  specifically to check node positions, port handle alignment, edge routing quality,
  wire overlaps, node overlaps, and wires passing through nodes. Run against the live
  dev server. Triggers on: "check the canvas", "verify the layout", "did the wires break",
  "audit the canvas", "test the new merge on the canvas".
---

# canvas-audit

Playwright tool that extracts VueFlow canvas state from the live DOM and reports layout
quality problems that are invisible to unit tests.

## Location

```
SignalCanvasFrontend/tools/canvas-audit/index.ts
```

## Usage

The dev server must be running (`npm run dev` or `npm run tauri:dev`).

```bash
# Capture against the default worship-venue example (best for layout testing)
npx tsx tools/canvas-audit/index.ts \
  --url "http://localhost:5173/#/canvas/worship-venue" \
  --out .canvas-audit

# Capture a specific patch file (load it first via MCP set_patch, then audit)
npx tsx tools/canvas-audit/index.ts \
  --url "http://localhost:5173/#/canvas" \
  --out .canvas-audit
```

Output:
- `.canvas-audit/report.json` — structured report (all metrics + findings)
- `.canvas-audit/nodes/<id>.png` — zoomed screenshot per node
- `.canvas-audit/full-canvas.png` — full canvas viewport screenshot
- `.canvas-audit/regions/<label>.png` — cropped screenshots of problem areas

## Reading the Report

```jsonc
{
  "nodeCount": 3,
  "edgeCount": 4,
  "routedEdgeCount": 4,          // edges that have actual route points
  "alignmentErrorCount": 0,       // wire endpoints not aligned to port handles
  "parallelOverlapCount": 0,      // parallel wires too close together
  "wireCrossingCount": 2,         // wires that cross each other
  "nodeOverlapCount": 0,          // nodes physically overlapping
  "wiresThroughNodeCount": 0,     // wires passing through a non-endpoint node
  "nodesOffViewportCount": 0      // nodes outside visible canvas area
}
```

**Severity thresholds (alignment errors):**
- `minor` — ≤ 2 flow px off (ignore)
- `moderate` — 2–6 flow px off (investigate)
- `severe` — > 6 flow px off (fix required)

## What to Check After a Merge

1. Run the audit against `worship-venue` (has nodes + edges, good baseline)
2. Check `alignmentErrorCount` — should be 0 severe
3. Check `parallelOverlapCount` — parallel wires need ≥ 10 flow px separation
4. Check `wiresThroughNodeCount` — always 0
5. Check `nodeOverlapCount` — always 0
6. Open `full-canvas.png` to visually confirm layout looks reasonable
7. If any counts > 0, open the relevant region screenshots for detail

## Interpreting Findings

| Finding | Likely cause | Where to look |
|---------|-------------|---------------|
| `alignmentErrors` (severe) | Port handle IDs changed, or ELK edge routing off | `elk.ts` sourceHandle/targetHandle |
| `parallelOverlaps` | Bundling regression — multiple edges on same route | `buildVfEdges` in `useCanvasPipeline.ts` |
| `wiresThroughNodes` | ELK not treating nodes as obstacles | ELK layout options |
| `nodeOverlaps` | fcose placement failure or locked position conflict | `fcosePlace.ts` |
| `nodesOffViewport` | fitView not firing, or seed position out of bounds | `relayoutIncremental` in `useCanvasPipeline.ts` |
| `routedEdgeCount` < `edgeCount` | Some edges have no route points | Check edge bundling in `buildVfEdges` |

## Quick Sanity Check (no findings expected)

After loading a patch via MCP `set_patch`, a clean canvas should show:
- All counts 0 except `wireCrossingCount` (some crossings are unavoidable in dense graphs)
- `routedEdgeCount` == `edgeCount`
- Every node in `nodes` array has a `screenshotFile`
