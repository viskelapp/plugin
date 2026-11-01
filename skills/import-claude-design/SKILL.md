---
name: import-claude-design
description: Import a claude.ai Design canvas (a Design artifact or its export folder) into Viskel as a new design. Use when the user asks to bring, move, import or upload a claude.ai Design, a canvas.json export or .dc.html boards into Viskel.
---

# Import a claude.ai Design canvas

## 1. Find the canvas

The source is a folder or artifact with `canvas.json` at its root (or under `project/`). Read `canvas.json`: it lists the boards (`.dc.html` files), their pages and positions.

If `viskel` is on PATH (`command -v viskel`), its server is https://app.viskel.com (`VISKEL_URL`, else `server` in `~/.config/viskel/cli.json`) and the export is a local folder, `viskel import <exportDir> --name "<name>"` uploads it in one step; skip to step 4. Otherwise use `import_design` as below.

## 2. Collect the files

Build the `files` list for `import_design`:

- `canvas.json` and every `.dc.html` it lists: `{path, content}` with the text as it is.
- Images: every `/_blob/<id>` the boards use, as `assets/<id>.<ext>` with `{path, base64}`. A blob with no file stays a dead `/_blob/` link and comes back as a warning.
- A `ds/` folder, if there is one: its files with their paths (text as `content`, binary as `base64`).

Keep the paths as in the canvas. Each file may be at most 15 MB; if one is larger, tell the user which.

## 3. Import

`import_design` with a short `name` (it becomes the design key and must be new) and `files`; add `space` when the user has more than one. It returns the new key, page and board counts, `warnings` and a `validate` summary.

Report every warning to the user as it is: skipped files not in `canvas.json`, missing assets.

## 4. Check

1. `validate` the new design. Fix only what is clearly broken by the import; leave the rest to the user.
2. `snapshot` the first board of each page and look at it.
3. `get_design` and give the user the viewer link of each page, with the board count.
