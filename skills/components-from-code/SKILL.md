---
name: components-from-code
description: Port a product's components into a Viskel design system from its code - primitives as component families with examples, and its compositions as patterns, layouts and pages built from those families. Use when the user asks to bring in, port, import or rebuild components, a component library, ui/ folder, patterns or page templates from a repo into Viskel.
---

# Bring in components and patterns from code

Workflow for porting what a product is built from into a Viskel design system. The Viskel server's own instructions and `update_design_system`'s description hold the rules, the family file format and the pattern fields; this skill is the order of work. Tokens and the identity guide come first, with `/viskel:design-language`.

## 1. Connect and check the system

1. Call `list_design_systems` and pick the system the user named. If the Viskel tools are missing or answer with an auth error, sign in as `/viskel:design` step 1 says.
2. `get_design_system(system, "checklist")`. If tokens, identity or `components.base` are not done, run `/viskel:design-language` first: components are styled only from the system's tokens.
3. Read `get_design_system(system)`, `components` and `patterns` so you know what exists and do not port it twice.

## 2. Inventory

List the component folders (`components/ui`, `components/`, `resources/js/components`, `src/components`) and the page and layout files. For each item, count where it is used (grep its import or tag across pages). Sort it into:

- **Family:** a primitive the product composes everything else from (button, input, select, checkbox, badge, card, dialog, table, dropdown menu, tabs, tooltip).
- **Pattern:** a recurring composition of primitives (a form with its errors, an empty state, a confirm dialog, a filter bar, a toast after an action).
- **Layout:** a screen skeleton with named regions (sidebar with content, list with detail, settings page).
- **Page:** a whole screen people use most (an index, a record, a settings screen).

Skip what no page uses. Tell the user the inventory before porting when they are there to answer; otherwise go in usage order and say so in the report.

## 3. Families

For each family, most used first:

1. Read its source and how pages use it: the variants, sizes and states that really occur.
2. Write `components.<family>` with `file` (its parts), `mirrors` (the real component), `references` (what you read) and `examples`: one per variant and state that occurs, with real content from the product, never lorem ipsum.
3. Look at the returned image in every theme and at the findings. Fix every finding and render error, then write again until both are clean. Do not move on with a broken family.

## 4. Patterns, layouts and pages

Start with three or four of each group, the most used; not a gallery.

1. Write `patterns.<name>` with `group` (pattern, layout or page), `description` and `whenToUse`.
2. `states`: one per situation the product shows (default, focus, field error, form error, submitting, success, empty), each markup made only of the system's `ui-*` tags. A tag the system lacks is a missing family: port it first.
3. `rules`: at most two do/don't pairs from how the product uses it. `copyRules`: the real wording conventions from the product's strings (error messages, button labels).
4. Layouts and pages: `regions`, each with how it behaves per screen size. `patterns`: the patterns it is built from.
5. Read the findings and fix them until there are none.

## 5. Check

`get_design_system(system, "checklist")` until the `components` and `patterns` steps are done: no family or pattern with findings, every family rendered clean since its last change. Ask the user before `publish_design_system`; publish only when they say so.

## 6. Report

End with a short report:

- **Ported:** families and patterns by group, with their source files.
- **Skipped:** items not ported and why (unused, duplicate, needs a runtime feature the format lacks).
- **Left open:** findings you could not clear, and runtime gaps you met.
- **Coverage:** ported items against the repo's component folders.
