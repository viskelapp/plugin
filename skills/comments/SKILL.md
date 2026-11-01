---
name: comments
description: Work through review comments on a Viskel design. Use when the user asks to address, fix, answer or go through comments or feedback on a Viskel design or board.
---

# Work through Viskel comments

## 1. Collect

`list_comments` for the design (add `page` and `board` to narrow it to one board). Each thread says what it is on in `at`; element comments carry the file, the current line and a snippet. `people` is who can be mentioned.

Sort the threads before touching anything:

- **Clear change**: the comment says what to do. Do it.
- **Needs a decision**: two reasonable answers, a product question, or a change that conflicts with another comment. Don't guess.
- **Not for you**: a question to a person, or already done.

## 2. Fix each clear thread

1. `read_file` the board (`lineNumbers: true` helps with element comments).
2. `edit_file` the smallest change that answers the comment. Fix any issues it returns.
3. `snapshot` the board and check the change looks right.
4. `reply_comment` on the thread with one or two sentences on what changed.
5. `resolve_comment` only when the thread is fully answered. Leave it open if you did part of it, and say which part is left.

Keep the `cursor` from each result and pass it as `since`; if someone else changed the same board meanwhile, read it again before the next edit.

## 3. Ask for decisions

For a thread that needs a decision, `reply_comment` with the options in a sentence each, and mention the commenter, or whoever the user names, as `@[Name](user:<id>)` with the id from `people`. They are told by mail. Don't resolve these.

## 4. Report

End with three short lists, each item naming the board and thread:

- **Changed**: what you fixed and resolved.
- **Needs a decision**: who you asked and about what.
- **Skipped**: and why.
