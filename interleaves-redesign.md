# Interleaves — Redesign Design Document
*Captured June 2026 — pre-coding planning session*

---

## Object Model

Three first-class objects. Use these names consistently throughout the UI.

### Score
A source document. Never practiced directly.
- One PDF (single or multi-page), or
- An assembled set of photos/images merged in Score Assembly
- Stores: source file(s), page order, any prep adjustments (crop, rotate)
- Has many Passages derived from it
- `score.sourceId` is the parent reference Passages point back to

### Passage
A practice item derived from a Score. What gets shuffled and timed.
- Linked to one or more Score pages via `passage.sourceId`
- Stores: page reference(s), annotation layer (permanent — fingerings, corrections), practice layer (session circles/boxes, not copied on duplicate)
- Single-page (common) or cross-page (two-tap sequential selection)
- Has optional `compasSettings` (BPM, beats, accent pattern)
- Has optional `measures` count (enables Blitz mode timing)
- Has frequency weighting (−1/0/+1)

### Session
A timed run through a set of Passages.
- Stores: which Passages, shuffle mode, loop count, rest settings, duration
- Future: session history, SM-2 spaced repetition weighting

---

## Workflow Phases

Linear but not locked — user can move backward freely.

```
Import → Score Assembly → Prep → Practice Setup → Practice → (Review)
```

### 1. Import
**Goal:** get files into the app with minimum friction.

- Single **Import** button opens the Import Window
- Import Window stays open; user accumulates items before committing
- Two re-triggerable buttons inside the window: **Add Files** and **Camera**
- Each addition appears as a thumbnail row (preview + editable name)
- Name field is optional at this stage — naming can be deferred to Prep
- **Named capture session**: before first camera shot, app optionally prompts for piece name → subsequent photos auto-append *pg. 1, pg. 2...* — simplifies Score Assembly later
- Single **Import** button inside window commits all pending items at once
- PDFs: auto-detected as multi-page, page count pre-populated
- Platform note: iOS and Android both return one photo per camera invocation (platform ceiling for web apps). Camera button must be easy to re-tap. Capacitor native wrap unlocks continuous camera session — a paid-tier differentiator.

### 2. Score Assembly *(optional)*
**Goal:** merge multiple files that belong to one piece into a single Score.

- Entered explicitly: select multiple items in library → **"Assemble into score"**
- Skipped entirely for single PDFs and single photos
- Interface: horizontal thumbnail strip, drag to reorder, confirm to merge
- Handles: loose photos, split PDFs, mixed file types
- Originals replaced by the assembled Score on confirm
- Future possibility: smart auto-assembly from named capture sessions (parked — if resolved cleanly, makes this step unnecessary)

### 3. Prep
**Goal:** clean up and structure the Score before marking passages.

- Crop, rotate, brightness/contrast per page
- Re-entry allowed; warning if annotations exist (prep changes are destructive to annotation layers)
- PDF Scores: lightweight page reader (PDF.js), decide page groupings if needed
- Assembled photo Scores: pages already ordered from Score Assembly

### 4. Practice Setup
**Goal:** define Passages from the Score.

- User opens a Score, sees it as a readable document
- Draws a circle or box around a passage
- Hits **"Save passage"** — annotation saves as a new Passage, clears from Score view
- Status line confirms ("Passage saved") — no badge on score for now, keeps score clean
- Repeat for each passage in the Score, or switch to another Score
- Cross-page passages: circle page 1 → **"Continues on next page"** → circle page 2 → **"Save passage"**
- Each Passage stores `passage.sourceId` pointing back to its parent Score

#### Annotation layers (Option A architecture)
- **Layer 1 — Score annotations**: fingerings, corrections, bowings. Permanent. Copies on duplicate. Drawn in Score Edit mode.
- **Layer 2 — Practice annotations**: circles, boxes indicating passages. Session-scoped. Does not copy. Drawn in Practice Setup mode.
- Layer is determined by *which mode you're in* — no manual toggle needed. Mode = layer selector.

### 5. Practice
**Goal:** timed interleaved practice session.

- Passages served in randomized order (shuffle-per-loop or full-shuffle)
- Compás auto-configures on item serve if `passage.compasSettings` is set
- Rest periods between passages (on by default)
- Frequency weighting (−1/0/+1) affects shuffle queue

#### Blitz Mode *(planned)*
- Each Passage served once
- Timer derived from musical data: `prep (2s) + count-in (1 measure) + play time (measures × beats ÷ BPM × 60)`
- Requires `passage.compasSettings` and `passage.measures` to be set
- Feels like performance simulation — distinct modality from repetition practice
- Maps directly onto retrieval practice research (strong Bulletproof Musician angle)

### 6. Review *(future)*
- Session history
- SM-2 spaced repetition weighting
- Repertoire structure view ("all passages from this piece")

---

## Compás Integration

### Item association
```js
passage.compasSettings = {
  bpm: 112,
  beats: 4,
  accentPattern: [true, false, false, false]
}
```
- Captured with one button during Practice Setup or Practice: **"Save Compás settings to this passage"**
- On serve: Interleaves writes to `compas_apply` localStorage key; Compás reads and configures
- If embedded: direct function call. If standalone: localStorage event bridge (pattern already established via shared `theme` key)

### Prerequisites for Blitz mode
- Count-in feature needed in Compás (1–2 measures before item starts)
- `passage.measures` field — manual entry, stable data, one-time cost, lives in Prep or Practice Setup

---

## Key Decisions Made

| Decision | Choice | Rationale |
|---|---|---|
| File copies | One copy per piece, virtual layers | No more duplicate confusion |
| Multi-page | Score Assembly module, not grouping | Cleaner separation of concerns |
| Annotation confusion | Mode determines layer, no toggle | "Almost invisible" per design philosophy |
| Import UX | Batch window, single commit | Reduces confusion between picker and trigger |
| Naming | Deferred to Prep, optional in Import | Keeps capture flow fast |
| Cross-page passages | Two-tap sequential | Single-page flow stays frictionless |
| Post-save feedback | Status line, not score badge | Clean score view |
| Passage verb | "Save passage" | Short, musical, self-explanatory |

---

## Open Questions

- **Smart auto-assembly**: can named capture sessions eliminate Score Assembly entirely? Parked — its own design problem.
- **Score view in Prep**: needs to feel like reading, not file management. PDF.js pagination UI needs design attention.
- **Compás architecture at merge time**: embedded vs. standalone affects how deep the integration can go. Decide before building Compás association feature.
- **Migration path**: existing DB has groups and single-blob annotations. V4 migration needs to flatten groups → multi-page Scores, split annotation canvases into layers. Design schema before building.

---

*This document should be updated as decisions are made. Coding starts after open questions are resolved.*
