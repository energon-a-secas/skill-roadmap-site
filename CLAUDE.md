# Skill Roadmap Builder — Project Guide

## Overview
A zero-dependency, zero-build static web app for building visual skill roadmaps.
Nodes represent skills/milestones, columns represent stages, lanes represent parallel paths within a stage.
Connections between nodes are typed bezier curves rendered on an SVG overlay.

Served via `python3 -m http.server 8777` or any static file server.

---

## File Structure

```
site/
├── index.html        # HTML shell (~160 lines) — structure only, no inline CSS or JS
├── css/
│   └── app.css       # All styles (~1070 lines)
└── js/
    └── app.js        # All application logic (~1940 lines)
```

### index.html
Pure markup. Contains: header, sidebar, canvas wrapper, SVG overlay, context menus, modal overlay, toast container.
All dynamic content is injected by `js/app.js` at runtime.

### css/app.css
Sections (in order):
- Reset + Design Tokens (CSS custom properties)
- App Shell, Header, Sidebar
- Column + Lane layout (`.lanes-wrapper`, `.lane`, `.lane-head-row`)
- Node cards (`.node`, `.node-lock`, `.port-*`)
- Context menus, Edge type menu
- Modal, Toast
- Connection mode cursor styles

### js/app.js
Sections (marked with `/* ===== ... ===== */` comments):
- **CONSTANTS** — `MARKER_ICONS`, `EDGE_TYPE_DEFS`, `LOCK_ICON_HTML`
- **DEFAULT STATE** — `defaultState()`, `blankState()`
- **STATE** — `let state`, `connectionMode`, `pointerDrag`
- **HELPERS** — `getNode`, `getColumn`, `nodesInLane`, etc.
- **MIGRATION** — `migrateState()` for backward-compatible saves
- **RENDER** — `renderAll`, `renderColumns`, `renderConnections`, `buildNodeElement`
- **CONNECTION MODE** — `handlePortClick`, `finishConnection`, `cancelConnectionMode`
- **TEMP CONNECTION LINE** — mousemove handler for rubber-band preview
- **EDGE TYPE MENU** — inline menu shown on line click
- **POINTER DRAG** — node reorder via drag (`onPointerDragStart`, `onPointerDragMove`, `onPointerDragEnd`)
- **CONTEXT MENU** — `showNodeContextMenu`, marker/color pickers, move-to-path
- **CRUD** — `addColumn`, `deleteColumn`, `addLane`, `deleteLane`, `addNode`, `deleteNode`
- **MODALS** — `showModal`, `closeModal`, `showConfirmModal`, `openAddColorModal`, `openAddMarkerModal`
- **SAVE / LOAD** — `saveState`, `loadState`, `saveJSON`, `loadJSON`
- **EXPORT** — `exportMarkdown`
- **BUTTON WIRING** — event listeners for all header buttons
- **INIT** — `(function init() {...})()`

---

## Data Model

```js
state = {
  title: string,
  columns: [
    {
      id: string,       // "col-X"
      name: string,
      lanes: [{ id: string, name: string }]  // min 1 lane per column
    }
  ],
  nodes: [
    {
      id: string,       // "n-X"
      columnId: string,
      laneId: string,
      row: number,      // vertical order within lane
      rowSpan: number,  // height multiplier (1–4)
      text: string,
      color: string,    // hex, must exist in colorLegend
      markers: string[],// keys from markerLegend
      locked: boolean   // manually locked
    }
  ],
  edges: [
    { id: string, from: nodeId, to: nodeId, type: EdgeType }
  ],
  colorLegend: { [hex]: label },
  markerLegend: { [key]: label }
}
```

`EdgeType` values: `"blocks"` | `"informs"` | `"enhances"` | `"enables"` | `"prepares"`

Persistence: `localStorage` key `"skillTreeData"`. `migrateState()` adds missing `lanes`/`laneId` for old saves.

---

## Key Patterns

### Rendering pipeline
`renderAll()` → `renderColumns()` + `renderConnections()` + `renderColorLegend()` + `renderMarkerLegend()`

`renderColumns()` rebuilds the entire column/lane/node DOM from scratch on every state change.
`renderConnections()` redraws all SVG bezier paths.

### Connection routing (`chooseRoute` in js/app.js)
- **Same lane** → bottom→top (vertical bezier)
- **Same column, different lane** → right→left (short horizontal arc)
- **Cross-column** → right→left based on column index order

Port positions: 4 sides (left, right, top, bottom) via `getPortPosition(nodeId, side)`.

### Port interaction (dual-mode)
Ports support both click-click and drag-to-connect:
- Short press (<6px movement) → `handlePortClick()` → click-click connection mode
- Long drag (≥6px) → starts `connectionMode` visually, release over node → `finishConnection()`

### Pointer drag (node reorder)
`pointerDrag` state tracks: `nodeId`, `ghost`, `dropColId`, `dropLaneId`, `dropBeforeId`.
Lane-aware: `updatePointerDropTarget()` finds `.lane` element under cursor.

### Locked state propagation
`updateLockedStates()` traverses `edges` to find transitively blocked nodes (any upstream `"blocks"` edge
from a locked or blocked node). Updates `.effectively-locked` CSS class and lock icon visibility.

---

## Improvement Backlog

### High priority
- [ ] **Undo/redo** — command pattern; wrap each mutation in a reversible action
- [ ] **Lane reordering** — drag lanes left/right within a column
- [ ] **Column reordering** — drag columns horizontally
- [ ] **Node search/filter** — filter nodes by text, color, or marker

### Medium priority
- [ ] **JS module split** — convert `js/app.js` to ES modules:
  - `js/state.js` — state, defaultState, blankState, migrateState, save/load
  - `js/render.js` — renderAll, renderColumns, renderConnections, buildNodeElement
  - `js/connections.js` — chooseRoute, getPortPosition, handlePortClick, etc.
  - `js/ui.js` — modal, toast, context menus, sidebar
  - `js/drag.js` — pointer drag for node reorder
  - `js/main.js` — init, button wiring
- [ ] **Minimap** — small overview panel for large diagrams
- [ ] **Export to PNG/SVG** — html2canvas or native SVG export

### Low priority / polish
- [ ] **Keyboard shortcuts** — Delete to remove selected node, Escape to cancel, Ctrl+Z undo
- [ ] **Multi-select** — shift-click to select multiple nodes and move/delete together
- [ ] **Node groups** — visual grouping of related nodes within a lane
- [ ] **Custom connection colors** — allow per-edge color override
- [ ] **Zoom + pan** — transform canvas for large diagrams
- [ ] **Responsive layout** — collapse sidebar on mobile, horizontal scroll on canvas
