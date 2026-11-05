---
name: sections
description: Organise a Viskel page into sections, label them (Draft, Ready for dev, Completed) and hand them off to development with Jira links. Use when the user asks to group boards into flows, plan sections, mark work ready for dev, or link boards to Jira issues in Viskel.
---

# Sections and handoff in Viskel

## 1. See what is there

`list_sections` (`design`, `page`) returns the sections with their labels and boards, the planned sections, and the design's `statuses`: the label list (defaults `draft`, `ready-for-dev`, `completed`). `list_boards` returns all boards of the page; the ungrouped ones are those not listed in any section from `list_sections`.

## 2. Group by flow

- One section per user flow or feature ("Checkout", "Onboarding"), not per screen type.
- Existing boards: `add_section` with `name` and `boards`; it fits the area around them.
- Work not drawn yet: `plan_section` with a `name` (and `labels`), so it shows as planned.
- Rename, move (carries its boards) or resize with `update_section` (`id` from `list_sections`).

## 3. Label

Labels are what the canvas shows. Set them with `update_section` `labels` (the whole list of ids, `[]` clears). Add or rename labels for the whole design with `set_section_statuses`, sending the whole list.

Before labelling a section `ready-for-dev`, for every board in it:

1. `validate` the page: no errors.
2. `snapshot` each board and look at it.
3. `list_comments` for each board: count the open threads. Open threads block "Ready for dev" unless the user says otherwise; list them instead.

## 4. Jira

- The design needs a Jira base URL once: `update_design` with `jiraBaseUrl` (only a workspace owner can set it; ask the user for the URL, e.g. https://yourcompany.atlassian.net).
- `link_jira` (`design`, `page`, `name`, `keys`) links a board to issue keys. Use your Jira tools, if any, for the issues themselves.
- `find_boards` with `jira` finds the boards linked to an issue, in one design or all.

## 5. Handoff summary

End with one block per section: name, label, its boards with their Jira keys, open comment count, and the page viewer link from `get_design`. Name anything that kept a section from "Ready for dev".
