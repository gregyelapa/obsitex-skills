# Lists

Bulleted and numbered lists, nested to any depth, plus Obsidian task lists.

> The general rules are in `obsitex-conventions.md`. This file only gives the list-specific
> cases.

## Syntax

```markdown
- First point
- Second point with **bold**
    - Sub-point a
    - Sub-point b
- Third point

1. Step one
2. Step two
```

- Bulleted: the line starts with `- `, `* ` or `+ `
- Numbered: the line starts with `1. ` (or `1) `)
- **Nesting** through indentation — one tab or about four spaces per level
- Items take the usual inline formatting: `**bold**`, `*italic*`, `` `code` ``, `$maths$`,
  citations, wikilinks

Becomes `\begin{itemize}` / `\begin{enumerate}`; sub-points become nested lists.

**A list needs no blank line before it** (since 19.10.2026). It may follow a sentence
directly, as in Obsidian:

```markdown
Three cities took part:
- Zurich
- Basel
```

This works only where Obsidian also shows a list: a bullet followed by text, or a numbered
item that starts at `1.` (or `1)`). A line such as `2. Step two` or `2026. was a year` directly
under a sentence stays part of the sentence's paragraph, in Obsidian and in the PDF. If a
numbered list must start at another number, the user's Obsidian shows it as text too, so the
two agree.

## Task lists

Obsidian task lists reach the PDF as printed boxes:

```markdown
- [ ] Write chapter 3
- [x] Literature reviewed
    - [ ] One sub-point still open
```

An open item becomes an empty box ☐, a done one a crossed box ⊠ — the same information the
checkbox carries in Obsidian. Nesting and inline formatting work as in any list.

**Only `[ ]` and `[x]` are boxes.** Anything else in the brackets — for example `[/]` from an
Obsidian task plugin — stays as literal text and gets an ordinary bullet. A real square
bracket at the start of an item (`- [Note] …`) is safe: it is not turned into a box.

The box symbols come from `amssymb`, which the standard preamble already loads. **No new
package.**

## Traps

All measured, not inferred: the line and heading traps on 14.08.2026, the blank lines and
numbers on 23.10.2026.

**One item = one line.** A continued line without a list marker *is* appended to the item —
but with a `\\` in front of it, the same forced break as in a paragraph:

```latex
\item First item that runs on\\
and continues on this line
```

The paragraph rule therefore applies inside lists too. Do not wrap items to a column width.

**Blank lines and numbers: the PDF follows Obsidian's reading view** (since 23.10.2026, measured
case by case against Obsidian's own renderer). Until then a blank line restarted the count; it
no longer does. What holds now:

- A blank line between two items does not end the list, however many blank lines there are.
  `1. 2. ⏎⏎ 3. 4.` is one list from 1 to 4.
- Two lists that both start at `1.`, with only blank lines between, are **one** list from 1 to
  4, as in Obsidian. Two separate lists need text between them.
- A numbered list starts at the number its first item carries. A paragraph between items
  therefore does not reset the count when the next item says `3.`:

```markdown
1. Alpha
2. Beta

A paragraph between them.

3. Gamma          ← prints as 3.
4. Delta
```

- A list goes on only with the same kind of marker: `- a ⏎⏎ * b` and `1. a ⏎⏎ 1) b` are two
  lists, `1. a ⏎⏎ - b` too. A sub-item after a blank line (indented) stays a sub-item.

**Two cases still differ (N100).** A paragraph indented under an item (`    text` after a blank
line) belongs to that item in Obsidian but prints as an ordinary paragraph between the items;
the numbers after it still run on. And a sub-list that starts at a number other than 1 right
under an item line (`1. a ⏎    3. b`) is plain text in Obsidian but a sub-list in the PDF.
Advise against both: put the paragraph into the item's own line, start a sub-list at 1.

**A `#` heading directly under a list item is swallowed by the list** and does not become a
heading. This is the one place where a blank line before a heading is still required.

**Indent sub-points consistently.** Mixed indentation (tabs on one line, spaces on the next)
can confuse the nesting.
