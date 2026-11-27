---
name: components-from-code
description: Port a product's components into a Viskel design system from its code - primitives as component families with examples, and its compositions as patterns, layouts and pages built from those families. Use when the user asks to bring in, port, import or rebuild components, a component library, ui/ folder, patterns or page templates from a repo into Viskel.
---

# Bring in components and patterns from code

Workflow for porting what a product is built from into a Viskel design system. The Viskel server's own instructions and `update_design_system`'s description hold the rules, the family file format and the pattern fields; this skill is the order of work. Tokens and the identity guide come first, with `/viskel:design-language`.

## 1. Connect and check the system

1. Call `list_design_systems` and pick the system the user named. If the Viskel tools are missing or answer with an auth error, sign in as `/viskel:design` step 1 says.
2. `get_design_system(system, "checklist")`. If tokens, identity or `components.base` are not done, run `/viskel:design-language` first: components are styled only from the system's tokens.
3. Read `get_design_system(system)`, `components` and `patterns` so you know what exists and do not port it twice. The `patterns` section also lists the fields a pattern takes.

## 2. Inventory

Find the component folders from `components.json` or the `@/` import alias (`components/ui`, `resources/js/Components/ui`, `src/components`), and the page and layout files. For each item, count where it is used, for example `grep -rhoE "@/[Cc]omponents/ui/[a-z-]+" resources/js | sort | uniq -c | sort -rn`, and the same for brand components (`@/[Cc]omponents/[A-Z][A-Za-z]+`) and layouts (`@/[Ll]ayouts/`). On a CMS or marketing site, count the blocks the pages render too: they are the patterns. Sort each item into:

- **Family:** a primitive the product composes everything else from (button, input, select, checkbox, badge, card, dialog, table, dropdown menu, tabs, tooltip), including brand components outside `ui/` that pages reuse.
- **Pattern:** a recurring composition of primitives (a form with its errors, an empty state, a confirm dialog, a filter bar, a toast after an action).
- **Layout:** a screen skeleton with named regions (sidebar with content, list with detail, settings page).
- **Page:** a whole screen people use most (an index, a record, a settings screen).

Skip what no page uses. Tell the user the inventory before porting when they are there to answer; otherwise go in usage order and say so in the report.

## 3. Families

For each family, most used first:

1. Read its source and how pages use it: the variants, sizes and states that really occur.
2. Write `components.<family>` with `file` (its parts; compose with families already ported, such as `ui-button` inside a dialog footer, and give a part a state prop such as open or highlighted when a pattern needs to show it), `mirrors` (the real component), `references` (what you read), `guidelines` (a list of short rules) and `examples`: one per variant and state pages use, with real content from the product (its strings or fallback text; where copy comes from a CMS, plausible product copy, said in the report), never lorem ipsum. Draw overlays (dialog, menu, select content, tooltip) open in place, not fixed or centred. A variant that only differs per screen size becomes an explicit variant plus a guideline.
3. Look at the returned image in every theme and at the findings. Fix every finding and render error, then write again until both are clean. Do not move on with a broken family.

## 4. Patterns, layouts and pages

Start with three or four of each group, the most used, unless the user sets the scope; not a gallery.

1. Write `patterns.<name>` with `group` (pattern, layout or page), `description` and `whenToUse`.
2. `states`: one per situation the product shows (default, focus, field error, form error, submitting, success, empty). Each state contains the system's `ui-*` tags; plain wrappers for layout are fine when they are styled only with the system's variables. A component the system lacks is a missing family: port it first, or, when the time you have is used up, draw it with token-styled HTML and list it under Left open. A situation with no component in it (a loading line) goes in `copyRules`.
3. `rules`: at most two do/don't pairs from how the product uses it. `copyRules`: the real wording conventions from the product's strings (error messages, button labels).
4. Layouts and pages: `regions: [{name, behaviour}]`, the behaviour saying how the region changes per screen size. `patterns`: the patterns it is built from.
5. A pattern write returns findings, not an image: read them and fix them until there are none. To see a state, open the pattern's page in the design system.

## 5. Check

`get_design_system(system, "checklist")` until the `components` and `patterns` steps are done: no family or pattern with findings, every family rendered clean since its last change. A done checklist means nothing ported has findings, not that everything is ported: the coverage in the report says how much is. If a finding contradicts the family's own records, or a family stays "not render-checked" after a clean render, report it as a server issue under Left open instead of writing again. Ask the user before `publish_design_system`; publish only when they say so.

## 6. Report

End with a short report:

- **Ported:** families and patterns by group, with their source files.
- **Skipped:** items not ported and why (unused, duplicate, needs a runtime feature the format lacks).
- **Left open:** findings you could not clear, missing families drawn as plain HTML, and runtime gaps you met.
- **Tokens and identity:** problems the port showed (a foreground that equals its background, a dim colour in dark mode); report them, do not change tokens here.
- **Coverage:** ported items against the repo's component folders.
