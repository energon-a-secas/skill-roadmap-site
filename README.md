# Skill Roadmap Builder

A web app for building visual skill trees and learning roadmaps the way you actually think about them. Stages as columns, skills as cards, relationships as typed arrows. No install, no build step, no dependencies — just open it.

## The idea

Most roadmap tools are either too rigid or too freeform. This one sits in the middle. You get structure (columns, lanes, ordered nodes) with enough flexibility to model anything from a learning path to a game skill tree. Connections between nodes carry meaning — whether something *requires*, *enables*, *relates to*, or *prepares* for something else — and the diagram updates visually when prerequisites aren't met.

## What you can do

**Columns and lanes** give you a grid to think in. Columns are stages or phases. Lanes inside a column let you run parallel tracks side by side without mixing them up.

**Nodes** are the skills, tasks, or milestones. Each one can have a color (to signal status), markers (to flag priority or state), a row span (if it needs more visual weight), and an icon for the game-style card layout.

**Connections** between nodes are typed. Requires, Relates, Enhances, Enables, Prepares — each has its own color and line style so you can read the diagram at a glance.

**Locking** works two ways. Nodes with unmet upstream requirements are automatically dimmed with a red border. You can also lock any node manually to mark it as blocked regardless of what's connected.

**Drag** to reorder nodes within a lane, or move them across lanes and columns entirely.

**Templates** to start from — a default skill roadmap or a game-style tiered skill tree with icon cards.

**Export** as JSON to save and reload, or as Markdown to share or document.

## Running it

```bash
python3 -m http.server 8777
```

Then open `http://localhost:8777`. Or just open `index.html` directly — no server required for basic use.

## Structure

```
index.html     markup shell
css/app.css    all styles
js/app.js      all logic (~2100 lines)
assets/icons   skill icons for the game card layout
```

State saves automatically to `localStorage`.

## Live

→ [ehq.cl](https://ehq.cl/)
