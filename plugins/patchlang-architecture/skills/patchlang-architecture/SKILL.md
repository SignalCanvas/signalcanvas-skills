---
name: patchlang-architecture
description: >
  Enforces the architecture boundary between PatchLang (.patch) and the JSON sidecar (.layout.json)
  in the SignalCanvas frontend. Use this skill whenever writing or modifying code that touches device
  data, connections, the canvas scene store, the emitter, the loader, the sidecar/layout JSON, or
  PlacedDevice/DeviceConnection types. Also use when adding new fields to device or connection types,
  generating IDs for devices or connections, or storing any data that might be device configuration.
  If you're touching canvasScene.ts, emitter.ts, loadFromPatchLang.ts, builderBridge.ts, or any file
  that references PlacedDevice, DeviceConnection, rfBand, rfActiveChannels, iemChannelModes,
  streamLabels, internalRoutes, internalBuses, installedCards, resolvedCardTypes, cardSlotGroups,
  or crypto.randomUUID — this skill applies.
---

# PatchLang Architecture Boundary

SignalCanvas uses two file formats. Each has a specific job — like how a live production separates the show file (mix data, routing, scene memories) from the surface layout (which fader is where on screen).

## The Two Files

**`.patch` (PatchLang)** is the show file. It stores everything about the system:
- Device definitions (templates)
- Device instances and their properties
- Connections between devices
- Internal routing (routes, buses)
- Channel labels and config
- Slot assignments (which cards are installed)
- RF configuration (band, active channels, IEM modes)
- Stream declarations
- Signals, flags, rings

**`.layout.json` (sidecar)** is the surface layout. It stores only how things look on screen:
- Device positions (x, y)
- Group boxes (position, size, color, label)
- Viewport (camera x, y, zoom)

That's it. Nothing else goes in the sidecar.

## Why This Matters

When device configuration leaks into the sidecar:
- **Data gets lost** when sharing `.patch` files (the sidecar doesn't travel with it)
- **Git diffs become unreadable** (JSON blobs vs. clean PatchLang lines)
- **Two sources of truth** means merge code to reconcile them, and bugs when they disagree
- **LLMs can't help** generate the correct JSON sidecar structure, but they can write PatchLang
- **Every sidecar field is tech debt** that must be migrated later

Think of it like writing your Dante routing on Post-it notes stuck to the console faceplate instead of in the show file. Works until you need to email the show file to the system tech at the next venue.

## Field Placement Rules

When you encounter these fields, they belong in PatchLang as instance properties — not the sidecar:

| Field | PatchLang Syntax | Builder Method |
|-------|-----------------|----------------|
| RF band | `rf_band: "G50"` (instance property) | `add_instance(json)` with property |
| RF active channels | `rf_active_channels: "4"` (instance property) | `add_instance(json)` with property |
| IEM channel modes | `iem_modes: "stereo,dual-mono,stereo"` (instance property) | `add_instance(json)` with property |
| RF channel labels | `config Instance { label RF_In[1]: "Pastor" { ... } }` | `set_rf_labels(instance, labels_json)` |
| Internal routes | `route Dante_In[1] -> Fader[1]` | `add_route(instance, from, from_ch, to, to_ch)` |
| Internal buses | `bus Main_LR { input: Fader[1..8]; output: Matrix_Out[1] }` | `add_bus(instance, bus_json)` |
| Installed cards | `slot MY_Slot[1]: MY16_AUD` | `set_slot(instance, slot, index, card)` |
| Stream labels | `stream SL_Dante { source: Stage_Left.Dante_Pri; ... }` | `add_stream(json)` |
| Channel labels | `config FOH { label Dante_Pri_In[1]: "Lead Vocal" }` | `set_label(instance, port, index, label, props)` |
| Resolved card types | NEVER store — derive from card template at compile time | N/A (delete this code) |
| Card slot groups | Derive from template `slot` syntax + card `meta` | N/A (derive at compile time) |

PatchLang instance bodies accept arbitrary key-value pairs, so `rf_band: "G50"` works today with zero language changes. Array-like data can use comma-separated strings: `iem_modes: "stereo,dual-mono,stereo"`.

## Identity System

Use PatchLang names as identity — not UUIDs.

PatchLang names everything: `FOH_Console`, `Dante_Pri_In[1]`, `StageBox_A.Dante_Out`. These are like cable labels — stable, readable, meaningful. Random UUIDs are like barcodes on cables: you need a lookup sheet to know what anything is, and if the sheet gets out of sync, cables get crossed.

**Rules:**
- Do not call `crypto.randomUUID()` for device or connection identity
- `instanceId` should be a PatchLang instance name (e.g., `"FOH_Console"`), not a UUID
- Connection IDs should be deterministic from source and target (e.g., `"connect_StageBox_Dante_Out_FOH_Dante_In"`), not random
- Port references should use PatchLang port names (e.g., `"Dante_Pri_In"`), not interface UUIDs

## The Builder API

The PatchLang Rust builder (in SignalCanvasLang, exposed via WASM) provides validated methods for all device mutations. Use these instead of direct store manipulation:

| Operation | Builder Method |
|-----------|---------------|
| Define a device type | `add_template(json)` |
| Place a device | `add_instance(json)` — accepts arbitrary key-value properties |
| Wire a connection | `add_connect(source, target, props)` |
| Install a card | `set_slot(instance, slot, index, card)` |
| Add internal route | `add_route(instance, from, from_ch, to, to_ch)` |
| Replace all routes | `set_routes(instance, routes_json)` |
| Add internal bus | `add_bus(instance, bus_json)` |
| Set channel label | `set_label(instance, port, index, label, props)` |
| Set RF labels | `set_rf_labels(instance, labels_json)` |
| Add stream | `add_stream(json)` |
| Add ring | `add_ring(json)` |
| Serialize to .patch | `format_program(handle)` |

The builder validates eagerly — if you pass invalid data (wrong port name, missing template), it returns an error immediately instead of silently corrupting state.

## Code Review Checklist

When reviewing or writing code, check for these patterns:

### Red flags in the emitter (`emitter.ts`)
If you see a new field being written to the sidecar's `DeviceLayoutEntry`:
```typescript
// BAD: device config leaking into sidecar
if (pd.rfBand) entry.rfBand = pd.rfBand
if (pd.someNewField) entry.someNewField = pd.someNewField
```
Ask: does this field affect signal flow or device configuration? If yes, it belongs in PatchLang via the builder, not in the sidecar.

### Red flags in the loader (`loadFromPatchLang.ts`)
If you see data being restored from the sidecar that isn't positional:
```typescript
// BAD: sidecar as source of truth for device config
if (layoutEntry?.rfBand) pd.rfBand = layoutEntry.rfBand
```
This data should come from PatchLang compilation, not the sidecar.

### Red flags: "fallback to sidecar" pattern
```typescript
// BAD: dual source of truth
if (!pd.internalBuses?.length && layoutEntry?.internalBuses) {
  pd.internalBuses = layoutEntry.internalBuses  // sidecar fallback
}
```
This means PatchLang can't fully express the data, so the sidecar fills the gaps. Fix the root cause: make PatchLang express the data, then remove the fallback.

### Red flags: UUID generation for identity
```typescript
// BAD: random identity
const instanceId = crypto.randomUUID()
const connectionId = crypto.randomUUID()
```
Use PatchLang names instead. Instance names come from the user or are generated deterministically. Connection IDs come from source + target port references.

## What Belongs Where — Quick Reference

| Data | .patch | .layout.json |
|------|--------|-------------|
| Device templates and instances | Yes | No |
| Connections and channel mappings | Yes | No |
| Internal routes and buses | Yes | No |
| Channel labels | Yes | No |
| RF band, active channels, IEM modes | Yes (instance properties) | No |
| Slot assignments | Yes | No |
| Stream declarations | Yes | No |
| Signals, flags, rings | Yes | No |
| **Device position (x, y)** | No | **Yes** |
| **Group boxes** | No | **Yes** |
| **Viewport (camera position, zoom)** | No | **Yes** |
| **UI-only display flags** | No | **Yes** (e.g., showChannelLabels) |
