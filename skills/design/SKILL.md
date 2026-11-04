---
name: design
description: Create or change a Viskel design (boards on the Viskel canvas). Use when the user asks to design, mock up, prototype or edit screens, pages or boards in Viskel, pastes a Viskel canvas, share or prototype link, or refers to "the selected element" in Viskel.
---

# Design in Viskel

Workflow for building and editing boards. The Viskel server's own instructions, the design's `CLAUDE.md` and the page's `PAGE.md` hold the rules; this skill is the order of work.

## 1. Connect

Call `list_designs`. If the Viskel tools are missing or answer with an auth error:

- Claude Code: run `/mcp`, pick the `viskel` server of this plugin and sign in in the browser.
- claude.ai or Claude Desktop: Customize > Plugins > Viskel > Connectors tab: Add (if it shows Not added), then Connect. On Team/Enterprise, an Owner must add it.

The user needs an existing account on https://app.viskel.com; signing in never creates one. Stop and say so if they have none.

## 2. Find the design

- A pasted link: `resolve_link` gives the design, page, board file and line. It takes canvas links (`…/<space>/designs/<key>/canvas#…`), share links (`…/s/<token>#page=…&board=…&node=…`, which Copy link now gives) and prototype links (`…/designs/<key>/play/<board>` or `…/s/<token>/play/<board>`).
- "The selected element" or "what I picked": `get_selection`.
- A name: `list_designs`, then `get_design` for pages, boards and viewer links.
- Nothing yet: `create_design` with a short `name`, a `title` and `pages` (`id`, `name`, `type`: website, app-screens, poster, social or wireframe). Ask which space when `list_designs` shows more than one.

## 3. Read, then plan

1. `read_file` the design `CLAUDE.md` and the page `pages/<page>/PAGE.md`. Follow them over anything here.
2. `list_boards` for the page, so new boards fit the existing names and rows.
3. Plan the boards before writing: one board per screen or state, names in PascalCase (`Checkout`, `CheckoutError`), sizes from `PAGE.md` (1440 desktop, 390 phone). Repeated parts (header, footer, card) become their own board and are reused with `<dc-import>`.
4. Tell the user the plan in a few lines when it is more than two boards.

## 4. Build

The format is in [references/board-format.md](references/board-format.md). Read it before the first board.

1. `add_board` (`design`, `page`, `name`, `w`, `h`; `newRow` for a new row, `interactive` for click-through prototypes, `expand: "fill"` for fluid web pages). It returns the skeleton's path and `version`.
2. Write the first full version with `write_file`, passing that `version`.
3. From then on change boards with `edit_file` (exact find/replace), not `write_file`. Read the file again if an `old_string` does not match; someone may have changed it.
4. Images and fonts: `upload_asset` (base64, up to 15 MB), referenced as `../assets/<name>`.

Real copy only: no lorem ipsum, no invented numbers. Use `[YOUR PRICE]` style placeholders where data is missing.

## 5. Check every board

After each board, not at the end:

1. Fix every error `edit_file` returns. Run `validate` (`design`, `page`) after `write_file` or structural changes.
2. `snapshot` the board (`design`, `page`, `board`; `fullPage: true` for long pages) and look at it. Fix overflow, clipping, broken images and render errors before moving on.

## 6. Working alongside others

- `claim_board` before a longer edit on an existing board; `release_board` when done. A claim warns others, it does not lock.
- Keep the `cursor` from each result and pass it as `since` on the next write. If `changedByOthers` lists your board, `read_file` before editing again.
- Something went wrong: `list_versions` for the file, then `revert_file` with the version to go back to. Nothing is lost.

## 7. Finish

1. For a new design, `set_design_thumbnail` with its main board.
2. `get_design` and give the user the viewer link of each page you changed, with one line per board on what it shows.
3. Say what is left open: placeholders, missing assets, states not drawn.
