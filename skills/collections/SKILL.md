---
name: collections
description: Create or extend a Viskel collection, a set of self-contained HTML artifact pages grouped in folders (docs, courses, reports, guides). Use when the user asks for a collection, artifact pages, a handbook, course or knowledge base in Viskel.
---

# Collections in Viskel

## 1. Open or create

- Existing: `get_collection` (`collection` key or title) for its settings and tree of folders and artifacts.
- New: `create_collection` with a short `name`, a `title`, a `description` that says who reads it and what for (it seeds COLLECTION.md), and a `look`:
  - `kit`: the Viskel style (default). Good for docs and reports.
  - `own`: each artifact brings its own styles. For pages with a strong look of their own.
  - `design-system`: the space's design system, when the pages should look like the product.

  Ask the user when the choice is not obvious.

## 2. Read COLLECTION.md

`read_file` `COLLECTION.md` (pass the collection key as `design`). It says how artifacts are written in this collection: look, CDN, theme, viewer state, links. Follow it.

## 3. Propose the tree

Decide what is one artifact and what is several, then show the user the folders and artifacts in a short outline before building. Keep a topic whole in one artifact unless it gets hard to navigate.

## 4. Build

1. `add_folder` (`collection`, `title`, `parent` folder id for nesting).
2. `add_artifact` (`collection`, `title`, `folder`) for each page. Pass `html` when you have it; otherwise write `artifacts/<id>/index.html` with `write_file` and the returned `version`.
3. Each artifact is one whole HTML document with its own layout and menu. Change it later with `read_file` and `edit_file`.
4. Link artifacts to each other with `viskel://<id>#<anchor>`, using ids from `get_collection`.
5. `update_node` renames or moves; `remove_node` trashes.

## 5. Finish

`get_collection` once more, then give the user the tree with one line per artifact. When you know the space slug (`list_designs` with that `space` returns it as `space.slug` on each design), add the link `https://app.viskel.com/<space>/collections/<key>`; otherwise give the collection key and title without a link.
