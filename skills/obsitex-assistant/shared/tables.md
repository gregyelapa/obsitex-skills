# Tables

GitHub-Flavored Markdown (GFM) tables — the same syntax Obsidian renders in reading view.
Obsitex turns them into a LaTeX `tabular`.

> The general rules — the three levels of a formatting answer, how a ` ```dds ` block
> behaves, and reading `00 Document Setup.md` before touching the preamble — are in
> `obsitex-conventions.md`. This file only gives the table-specific cases.

## Syntax

```markdown
| Material  | Amount | Price |
|:----------|:------:|------:|
| Screws    | 100    | 5 %   |
| Wood      | 2      | $10   |
```

- **First row** = column headers.
- **Second row** = the separator. It is mandatory — without it the block is not a table.
- **An empty line above the table is mandatory too.** A table directly under a line of text
  is not a table: Obsidian shows the pipes as text, and Obsitex appends the rows to the
  paragraph above, `|---|---|` included, each with a forced line break. No warning. Measured
  02.10.2026 against a control with the empty line, which converted to a normal `table`. It
  happens most easily when inserting at a cursor or a marker, see `cursor.md`.
- Colons in the separator set the alignment per column:
  `:---` left · `:--:` centred · `---:` right.
- Cells take normal inline formatting: `**bold**`, `*italic*`, `` `code` ``, citations,
  wikilinks.
- The column count comes from the header row. Missing cells are filled in empty; extra
  cells are dropped.

## Line breaks inside a cell

A cell cannot contain a blank line. To break a line inside a cell, use `<br>`, exactly as
Obsidian shows it:

```markdown
| Topic | Notes |
|---|---|
| Scope | First point<br>Second point |
```

`<br>` is the **one** HTML tag that works here. Everywhere else in the document HTML tags
land raw in the PDF, see `not-supported.md`. `<br><br>` gives an empty line between the two,
as in Obsidian.

**It works in every ordinary table, nothing else to do (since 23.10.2026).** The converter
turns such a cell into a small table of its own, top-aligned, in the alignment of its column.
No template change, no fixed widths, no package. Commands at the start of the cell (the marks
`` `\rowcolor{…}` ``, `` `\midrule` ``, `` `\endhead` ``) still act on the row. Measured with a
shaded heading row, three lines and a long table: 0 errors.

**A list inside a cell does not exist**, neither in Markdown nor in Obsidian: a cell holds
lines, not a list. For lines that look like one, put each point after a `<br>` and start it with
`•` (or `–`, or a number such as `1.` that the author types):

```markdown
| Topic | Points |
|---|---|
| Method | • First point<br>• Second point |
```

Obsidian and the PDF then show the same: each point on its own line. Prefer `•` over
`` `\textbullet{}` ``, because Obsidian shows the bullet instead of code. Three limits, say them
when they matter (measured 23.10.2026):

- **A long point wraps in Obsidian, in the PDF only with fixed column widths.** In an ordinary
  table it stays on one line and the table runs off the page (see "Traps").
- **Where it wraps, the second line starts under the bullet, not under the text**, in Obsidian
  as in the PDF.
- **No second level:** spaces at the start of a line are dropped, so `◦` cannot be indented.

**Until 23.10.2026 the break was lost without a word** in ordinary columns: Obsidian showed two
lines, the PDF one. A table that was switched to fixed `p{…}` columns back then for this reason
alone can go back to the normal template now; offer it, do not do it unasked.

**Fixed column widths are a separate wish:** a column whose long text should wrap within a set
width. Then swap the environment for that table, the same `dds` sandwich as everywhere else,
with the reset after it:

````markdown
```dds
{"tableEnvironmentText":"\\begin{table}[!htbp]\n    \\centering\n    \\begin{tabular}{|p{3cm}|p{6cm}|}\n        %tablebody%\n    \\end{tabular}\n\\end{table}\n\n"}
```
````

Ask for the widths, or say which you took. `%columnspec%` is deliberately gone, the alignment
markers cannot produce a `p` column. In such a table `<br>` becomes a paragraph break, which a
`p` column breaks as intended. For a long table use `longtable` with `p{…}` columns the same
way; it also lifts the footnote restriction under "Traps".

## What it produces

A LaTeX `tabular`. The exact look comes from four DDS keys, not from the converter:

| DDS key | Effect |
|---|---|
| `tableGridHorizontal` | draws the horizontal rules (`\hline`) |
| `tableGridVertical` | draws the vertical rules (`\|`) |
| `tableAlignment` | default alignment for columns without a `:` marker |
| `tableEnvironmentText` | the whole surrounding environment (table / centering) |

With a full grid the column spec becomes `{|l|c|r|}`. Full field catalogue: `dds.md`.

**Changing the look from here on:** a ` ```dds ` block before the table, and one after it to
put the value back, or every later table inherits the change:

````markdown
```dds
{"tableGridVertical": false}
```
````

Which values give which lines, and the look most users mean, is in the next section.

## Lines: four looks (since 23.10.2026)

The templates start with a **full grid**. Users ask for one of four looks. The Blueprints card
"Change the lines in tables" (`table-look`) spells them out in exactly these words:

| The user asks for | `tableGridHorizontal` | `tableGridVertical` | Also needed |
|---|---|---|---|
| a full grid | `true` | `true` | nothing |
| lines between the rows only, no lines between the columns | `true` | `false` | nothing |
| three lines: one above the table, one under the heading row and one at the bottom | `false` | `false` | `\toprule` and `\bottomrule` in `tableEnvironmentText`, and a mark in every table, see "Three lines" |
| no lines at all | `false` | `false` | nothing |

**"Horizontal lines only" is the second row: a line under every row.** If a user's own words
could mean the second or the third look ("only horizontal lines", "like in a journal"), ask
which one and name both. The third is the look of most published papers (the `booktabs` style).

### Three lines

The converter hands all rows to `tableEnvironmentText` in one piece (`%tablebody%`). So the
environment can draw a line above the table and one below it, but not the line between the
heading row and the first data row. That one comes from a **mark at the start of the first
cell of the first data row**, a LaTeX command in backticks, exactly like `\rowcolor` under
"Colour":

````markdown
```dds
{"tableGridHorizontal": false, "tableGridVertical": false, "tableEnvironmentText": "\\begin{table}[!htbp]\n    \\centering\n    \\begin{tabular}{%columnspec%}\n        \\toprule\n        %tablebody%\n        \\bottomrule\n    \\end{tabular}\n    \\caption{%caption%}\n    \\label{tab:%label%}\n\\end{table}\n\n"}
```

| Group | Count | Share |
|:------|------:|------:|
| `\midrule` A | 12 | 40 % |
| B | 18 | 60 % |
table: Groups by size
````

- **Build the environment from this vault's own `tableEnvironmentText`**, not from the
  example. Copy it out of `00 Document Setup.md` (or out of the last ` ```dds ` block above the
  table) and add only `\toprule` before `%tablebody%` and `\bottomrule` after it.
- **`\toprule`, `\midrule` and `\bottomrule` come from the `booktabs` package.** All three
  templates load it. Check the user's own preamble all the same (see "Packages"). If it is
  missing, add `\usepackage{booktabs}` with a `%` comment saying what it is for, and tell the
  user.
- **The block after the table puts back all three values** this vault had at that spot,
  character for character, as under "Placement".
- **Say to the author in one sentence:** Obsidian shows `\midrule` as code in the first data
  cell. That is expected, the PDF shows a line there.

Measured 23.10.2026 through the converter and pdflatex: 0 errors.

**How far the request reaches** decides where the values go and which tables get the mark:

| The request | The three values | The mark |
|---|---|---|
| one table | a block before it, a block after it | in that table |
| every table in one file | a block before the first table, a block after the last | in every table between the two blocks |
| every table in the document | in `00 Document Setup.md`, no blocks | in every table of the manuscript, **and in every table written from then on** (see "Fixed, in all three ways") |

**Leaving three lines means removing every mark** in the tables concerned, in the same edit. A
`\midrule` left in a table with any other look draws a second line under the heading row, and
in a full grid the column lines break off there (measured 23.10.2026). It still compiles, so
nothing warns.

**Together with the other table cases** (each measured 23.10.2026, 0 errors):

- **Shaded heading row:** `\rowcolor` stays in the first heading cell, `\midrule` goes into the
  first data cell. A thin white strip stays above and below the grey, because `booktabs` leaves
  room around its lines and the colour does not fill it. Say so in one sentence.
- **Line break in a cell:** `<br>` works as everywhere (see "Line breaks inside a cell"). A table
  with fixed `p{…}` columns takes that environment without the `|` and with `\toprule` and
  `\bottomrule` added.
- **Long table:** its own two templates and marks, see "Long tables". The heading row and both
  of its lines repeat on every page. The pages in between end without a line at the foot, which
  is usual for this look.
- **Pinned table:** nothing extra. Copying the vault's environment and changing only the
  placement letters (see "Placement") carries the two lines along.

## Placement: why a table moves

A table is a **float**. LaTeX does not print it at the spot in the source; it looks for a place
where the page breaks well. The author sees the table a page later, or at the top of the page
above the very sentence that announces it, and reads that as a bug. It is not. Explain the
mechanism before you change anything.

The templates set `[!htbp]`. What the letters mean and why exactly this combination was chosen:
`dds.md`, "Placement". Two facts settle most questions:

- **A float never travels backwards.** It moves down, or up to the top of the page it is
  already on, never to an earlier page.
- **A bare `[h]` is the one to avoid.** LaTeX cannot honour it, silently rewrites it to `[ht]`
  and warns once per float.

**Nailing one table down.** `[H]` from the `float` package does not negotiate. It needs a
` ```dds ` block before the table and a **second one after it** putting `tableEnvironmentText`
back, or every later table is pinned too:

````markdown
```dds
{"tableEnvironmentText": "\\begin{table}[H]\n    \\centering\n    \\begin{tabular}{%columnspec%}\n        %tablebody%\n    \\end{tabular}\n    \\caption{%caption%}\n    \\label{tab:%label%}\n\\end{table}\n\n"}
```
````

Two things to check and to say out loud when you do this:

- **`[H]` needs the `float` package. Read the user's own preamble before you assume it is
  there.** All three `obsitex-init` templates load it, but a hand-built or older
  `00 Document Setup.md` often does not (measured 29.08.2026). If it is missing, add
  `\usepackage{float}` with a `%` comment saying what it is for, and tell the user. Never
  silently — `obsitex-conventions.md`.
- A pinned table that no longer fits the rest of the page jumps to the next page whole and
  leaves the remainder of this one empty. That is the price of `[H]`.

**Build the second block from the value you actually found**, not from the example above. The
reset has to restore *this* vault's `tableEnvironmentText` character for character, so copy it
out of `00 Document Setup.md` (or out of the last `dds` block above the table) and change only
the placement letters.

**Do not try to keep a table below its text by dropping the `t`.** `[!hbp]` was measured and is
worse: floats keep their order, so one waiting table holds up every later one and they collect
at the foot of the page, far from the headings they belong to. If a table must not move, pin it.

**Going back to the standard** is the same move in reverse: one ` ```dds ` block setting
`tableEnvironmentText` back to `[!htbp]`, or removing the blocks that pinned it. Worth
offering before the manuscript is printed or handed in — gaps read worse than a table one page
on.

## Colour without giving up the Markdown table

**A LaTeX command inside a cell reaches the output verbatim**, because inline backticks are
passed through unescaped. So a table can be coloured and still be a real Markdown table —
no raw block needed.

**A grey header row** — `\rowcolor` at the start of the first header cell colours the whole
row. One backtick span, nothing else changes:

```markdown
| `\rowcolor{gray!25}` Material | Amount | Price |
|:---|:---:|---:|
| Screws | 100 | 5 % |
```

**A single cell** — `\cellcolor` in that cell. For a total, or when only some header cells
should be shaded:

```markdown
| `\cellcolor{yellow!30}` Total | `\cellcolor{yellow!30}` 220 |
```

Both verified through the converter and pdflatex: they compile clean and the colour appears.

**Prefer this over a raw block.** The author keeps a table they can edit, and Obsidian keeps
rendering it — only the one backtick span shows as code.

### A shaded heading row (since 23.10.2026)

The Blueprints card "Shade the heading row" (`table-colour`) names one of four colours, or the
user's own. Use these values:

| The user asks for | Value |
|---|---|
| light grey | `gray!25` |
| light blue | `blue!12` |
| light green | `green!12` |
| light yellow | `yellow!35` |
| another colour | the nearest `xcolor` name, mixed light (`!12` to `!35`). Say which value you took |

Light grey survives a black and white print best; the others turn into a similar grey there.
Measured 23.10.2026, all four 0 errors, the column lines stay visible.

**How far the request reaches** decides where the colour goes:

| The request | Where |
|---|---|
| one table | the mark `` `\rowcolor{…}` `` at the start of its first heading cell |
| every table in one file | the same mark in every table of that file, no ` ```dds ` block |
| every table in the document | **no marks:** `\rowcolor{…}` in `tableEnvironmentText` of `00 Document Setup.md`, right before `%tablebody%` (after `\toprule` where tables have three lines). Every table, and every table written later, gets the colour by itself |

For the whole document, the setting looks like this (the template's own environment with one
addition):

````markdown
"tableEnvironmentText": "\\begin{table}[!htbp]\n    \\centering\n    \\begin{tabular}{%columnspec%}\n        \\rowcolor{gray!25}\n        %tablebody%\n    \\end{tabular}\n    \\caption{%caption%}\n    \\label{tab:%label%}\n\\end{table}\n\n"
````

Measured 23.10.2026 with a full grid, with three lines and with no lines: 0 errors, the top line
stays, a table after a put-back block is plain again.

**Long tables are the exception: never put `\rowcolor` into a long table's template.** In a
`longtable` it lands right after the caption row and turns the first heading cell **black**,
text gone, no error (measured 23.10.2026). A long table keeps the marks instead, also when the
rest of the document is shaded by the setting: `` `\rowcolor{…}` `` in its first heading cell,
and in its copied heading row the same colour at the end of the mark (see "Long tables").

**Taking the colour away** means removing every mark in the tables concerned, or the
`\rowcolor{…}` from the setting, in the same edit. Where tables have three lines, a thin white
strip stays above and below the colour (see "Three lines"); say so in one sentence. Where the
colour comes from a mark, say that Obsidian shows it as code in the first heading cell.

## Packages: tables usually need none

The standard preamble already carries `array`, `booktabs`, `tabularx`, `longtable` and
`\usepackage[table,xcdraw]{xcolor}` — the last one is what provides `\rowcolor` and
`\cellcolor`. So most table wishes need **no** new package. That list describes the template,
not necessarily this vault; check the user's own preamble.

## Captions (since 29.08.2026)

A Markdown table CAN carry a caption. Write it on the line right after the table, opened
with `:`, `Table:` or `table:` and one space. This is Pandoc's spelling, not an Obsitex
invention.

```markdown
| Material | Amount | Price |
|:---------|-------:|------:|
| Screws   |    100 |   5 % |
table: Overview of the research design
```

That yields `\caption{…}` and `\label{tab:overview_of_the_research_design}`, so the table
appears in the List of Tables. **A raw LaTeX block is no longer needed for a caption.**

Four things worth knowing:

- **A blank line between table and caption is allowed but not required.** Without one the
  caption sits closer to the table in Obsidian's preview, which reads better. With one it is
  also valid Pandoc. Both produce the same LaTeX.
- **Position binds, not the pattern.** A line starting with `: ` anywhere else stays ordinary
  text. Only a line directly after a table becomes a caption.
- **The caption runs through inline rendering.** `**bold**`, `[[links]]` and `[@cite]` work
  in it. The label is built from the raw text, so markup never lands inside `\label`.
- **No caption means no `\caption{}` either.** The empty value takes the whole command with
  it. A table without a caption looks exactly as it did before.

### Where the caption stands (since 23.10.2026)

The templates put the caption **below** the table. Many guidelines want it **above**, APA
above with its own look. It is a style of the **whole document**: one table with the caption on
top and the next with it below would look wrong. The Blueprints card "Give a table a caption"
(`table-caption`) says it in one sentence ("Put every table caption in my document …").

| The request says | `tableEnvironmentText` in `00 Document Setup.md` | Preamble |
|---|---|---|
| below its table | the template's own: `\caption` and `\label` after `\end{tabular}` | nothing |
| above its table | `\caption{%caption%}` and `\label{tab:%label%}` moved before `\begin{tabular}`, `\centering` stays | nothing |
| above its table in APA style | the same, **without** `\centering`: the table stands flush left, like its caption | `\captionsetup[table]{labelsep=newline, labelfont=bf, textfont=it, justification=raggedright, singlelinecheck=false}` right after `\usepackage{caption}`, with a `%` comment |

````markdown
"tableEnvironmentText": "\\begin{table}[!htbp]\n    \\centering\n    \\caption{%caption%}\n    \\label{tab:%label%}\n    \\begin{tabular}{%columnspec%}\n        %tablebody%\n    \\end{tabular}\n\\end{table}\n\n"
````

- **A request that names no position leaves it as it is.** Giving one table a caption never
  moves the captions of the others.
- **Build from this vault's own environment** and move only the two lines (for APA, drop
  `\centering` too). The `caption` package comes with all three templates; check the user's own
  preamble all the same and add `\usepackage{caption}` with a comment if it is missing.
- **The put-back blocks in the notes** (pinned tables and the like) restore the old environment.
  Bring them in line, see `obsitex-conventions.md`.
- **Long tables** keep their caption above in any case (see "Long tables"); with "below" say so
  in one sentence. In APA style a long table also stands flush left:
  `\begin{longtable}[l]{%columnspec%}` in both of its templates, or it stands centred under a
  caption flush left.
- **Figures stay as they are:** `[table]` limits the APA line to tables.

Measured 23.10.2026 through the converter and pdflatex: all three positions, the APA line in the
preamble with a normal and a long table, a figure next to them, 0 errors, the List of Tables
correct.

Two tables sharing one caption get `_2`, `_3` on the label. Table labels live in their own
pot, separate from figure labels: `tab:` and `fig:` are different namespaces in LaTeX.

**Referring to a table** works like referring to a heading, since 29.08.2026:

```markdown
The split is shown in [[#Overview of the research design]].
```

That becomes `\vref{tab:overview_of_the_research_design}`. Three forms exist:

| Written | Result |
|---|---|
| `[[#Caption]]` (same note) | `\vref{tab:…}` |
| `[[Note#Caption]]` | the same, across notes |
| `[[#Caption\|the split]]` | `\hyperref[tab:…]{the split}` |

**A bare `[[Caption]]` does NOT work, on purpose.** That namespace belongs to note names. Point
at the caption after a `#`, exactly as you would at a heading.

**A table without a caption cannot be referred to at all** — no caption, no label. If a user
wants a reference, they need a caption first.

**A block identifier works on a table too** since 02.09.2026: `^id` after the table,
`[[#^id]]` to reference it. It needs a caption all the same, because the label comes from
there. Details: `links.md`.

**What the reader sees is "table 3 on the preceding page"**, not a bare number: the type
word comes from `cleveref`, the position from `varioref`, and the position is dropped when
the target is on the same page. Anatomy and the language trap: `links.md`.

**In Obsidian this link looks dead — that is expected, not a bug.** Obsidian resolves a `#`
only to **headings** (and `#^` to block ids); it knows nothing about captions. So the link is
painted as unresolved and hovering it says "… not found". **Obsitex still converts it
correctly** and the reference appears in the PDF. Measured 01.09.2026.

Two consequences, both silent, and worth saying out loud when a user writes one of these:

- **No click while writing.** The reference cannot be checked before converting.
- **Obsidian does not follow a renamed caption.** Change the caption and the link breaks
  without a warning; it shows up in the PDF as plain text where a number should be.

So when you edit a caption, **search the vault for links pointing at it** and update them in
the same breath. Nothing else will.

**Images work the same way since 01.09.2026**: `[[#Caption]]` reaches a figure and produces
`\vref{fig:…}`. An image without a caption is reachable too, because the converter makes its
file name the caption. Details in `images.md`.

A raw LaTeX block is still the answer for a table that needs a structure the DDS template
cannot express (`tabularx` for one table only, `multirow`, a sideways table). Not for a
caption.

## Long tables: more than one page (since 18.09.2026)

A normal table cannot break across pages. A table that may run over more than one page needs
`longtable` instead. **Use exactly one of the two templates below, never build your own.**
Which one depends on a single thing: does the table have a caption?

**With a caption.** The caption goes **above** the table. Below would be wrong here: the List
of Tables would then give the page where the table ends, not where it starts (measured).

````markdown
```dds
{"tableEnvironmentText": "\\begin{longtable}{%columnspec%}\n    \\caption{%caption%}\\label{tab:%label%}\\\\\n    %tablebody%\n\\end{longtable}\n\n"}
```
````

**Without a caption.** Do not use the first template with the caption left out: the row end
after the caption stays behind and prints an empty row above the table. And a `longtable`
counts itself as a table even without a caption, so every later table would carry the wrong
number ("Table 2" for the first captioned one). This template takes the count back:

````markdown
```dds
{"tableEnvironmentText": "\\addtocounter{table}{-1}\n\\begin{longtable}{%columnspec%}\n    %tablebody%\n\\end{longtable}\n\n"}
```
````

Both measured 18.09.2026 through the converter and pdflatex: 60 rows over two pages, 0 errors,
the List of Tables and the numbering of a later table correct.

**Where tables have three lines** (see "Lines: four looks"), take these two instead. They add
`\toprule` and `\bottomrule` and are otherwise the same. If this one table alone gets three
lines, its block also sets `tableGridHorizontal` and `tableGridVertical` to `false`, and the
block after it puts all three values back.

````markdown
```dds
{"tableEnvironmentText": "\\begin{longtable}{%columnspec%}\n    \\caption{%caption%}\\label{tab:%label%}\\\\\n    \\toprule\n    %tablebody%\n    \\bottomrule\n\\end{longtable}\n\n"}
```
````

````markdown
```dds
{"tableEnvironmentText": "\\addtocounter{table}{-1}\n\\begin{longtable}{%columnspec%}\n    \\toprule\n    %tablebody%\n    \\bottomrule\n\\end{longtable}\n\n"}
```
````

Measured 23.10.2026, 60 rows over three pages each, 0 errors, one List of Tables entry, a later
table numbered correctly. The marks for three lines are in the table below.

### Repeating the header row on every page

A long table repeats its header row on every following page, **but only if you mark where the
header ends.** The converter hands all rows over in one piece, so the template cannot do it. The
mark goes into a cell, as a LaTeX command in backticks, exactly like `\rowcolor` under "Colour".
**Always set it when you write a long table.** Which marks, again depends only on the caption.

**Without a caption: one mark.** The first data row starts with `` `\endhead` ``:

```markdown
| Criterion   | Description   | Source   |
|:------------|:--------------|:---------|
| `\endhead` Criterion 1 | Description 1 | Source 1 |
| Criterion 2 | Description 2 | Source 2 |
```

**With a caption: the header row twice.** With only the one mark, the caption becomes part of
the repeated head: it prints on every page and the List of Tables lists the table once per page
(measured: three entries for a table over three pages). So write the header row a second time
as the first data row, marked `` `\endfirsthead\hline` ``, and mark the first real data row with
`` `\endhead` ``:

```markdown
| Criterion   | Description   | Source   |
|:------------|:--------------|:---------|
| `\endfirsthead\hline` Criterion | Description | Source |
| `\endhead` Criterion 1 | Description 1 | Source 1 |
| Criterion 2 | Description 2 | Source 2 |
table: Selection criteria
```

The first page gets caption and header, every following page the header alone, and the List
of Tables one entry.

The mark in the copy depends on the vault, so build it from what you read:

| In this vault | Mark on the copied header row |
|---|---|
| `tableGridHorizontal: true` (the templates) | `` `\endfirsthead\hline` `` |
| `tableGridHorizontal: false` | `` `\endfirsthead` `` |
| header row shaded with `\rowcolor{…}` | the same `\rowcolor{…}` added at the end of the mark, e.g. `` `\endfirsthead\hline\rowcolor{gray!25}` `` |
| three lines | `` `\midrule\endfirsthead\toprule` ``. And `` `\endhead` `` becomes `` `\midrule\endhead` ``, with or without a caption |

The copy carries the header text **character for character**, only the mark in front differs.
All of this measured 18.09.2026 through converter and pdflatex, 60 rows over two pages: header
repeated (shaded and unshaded, with and without horizontal rules), one List of Tables entry, 0
errors, no "multiply defined" warnings.

**Say two things to the author, in one sentence each:**

- Obsidian shows the marks as code in the first cells, and with a caption the header row twice.
  That is expected; the PDF shows each once.
- **The marks work only inside the long table.** If the `dds` block above is ever removed, the
  marks have to go too, or the PDF stops with a LaTeX error. A new row goes below the
  `` `\endhead` `` row, never above it.

### Four things that go with it

- **The sandwich, as always:** the block goes directly before the table, and a second block
  after it puts `tableEnvironmentText` back. Build that second block from the value this vault
  actually uses (`00 Document Setup.md` or the last `dds` block above), character for character,
  as under "Placement".
- **Adding a caption later means swapping the template and the marks.** The template without a
  caption takes the count back; with a caption added, two tables would share a number. Change
  the block before the table, add the copied header row with its mark, all in the same edit.
  Removing a caption is the same move in reverse.
- **A long table that becomes short** (the author says it no longer needs to break): remove the
  `dds` blocks **and** every mark in the same edit.
- **Grid, alignment and a shaded header row work as in a normal table.** `longtable` also lifts
  the footnote restriction under "Traps". It is already loaded in the standard preamble; check
  the author's own preamble all the same (see "Packages").

## A table too wide for the page (since 23.10.2026)

Obsidian fits a table to the window, the PDF does not: an ordinary column never wraps, so a
table with long cells runs off the right edge of the page. Nothing warns. The Blueprints card
"Make a wide table fit" (`wide-table`) names one of three ways, always for **one** table, the
one the author marked (only the author sees which table runs off; you do not see the PDF):

| The request says | What you write |
|---|---|
| let the columns with long text wrap | `tabularx` over the text width: the columns with long text become `>{\raggedright\arraybackslash}X`, the others keep their `l`/`c`/`r` |
| put the table on a page of its own in landscape format | `\begin{landscape}` … `\end{landscape}` around the whole `table` environment (package `pdflscape`) |
| shrink the whole table to the text width | `\resizebox{\textwidth}{!}{…}` around the `tabular` (package `graphicx`) |

Each is a ` ```dds ` pair around that one table, built from this vault's own
`tableEnvironmentText` (caption position, lines and the rest stay as they are). Examples from
the template's environment:

````markdown
```dds
{"tableEnvironmentText": "\\begin{table}[!htbp]\n    \\centering\n    \\begin{tabularx}{\\textwidth}{|l|>{\\raggedright\\arraybackslash}X|>{\\raggedright\\arraybackslash}X|r|}\n        %tablebody%\n    \\end{tabularx}\n    \\caption{%caption%}\n    \\label{tab:%label%}\n\\end{table}\n\n"}
```
````

````markdown
```dds
{"tableEnvironmentText": "\\begin{landscape}\n\\begin{table}[!htbp]\n    \\centering\n    \\begin{tabular}{%columnspec%}\n        %tablebody%\n    \\end{tabular}\n    \\caption{%caption%}\n    \\label{tab:%label%}\n\\end{table}\n\\end{landscape}\n\n"}
```
````

````markdown
```dds
{"tableEnvironmentText": "\\begin{table}[!htbp]\n    \\centering\n    \\resizebox{\\textwidth}{!}{\\begin{tabular}{%columnspec%}\n        %tablebody%\n    \\end{tabular}}\n    \\caption{%caption%}\n    \\label{tab:%label%}\n\\end{table}\n\n"}
```
````

- **Wrapping writes the column spec by hand**, so `%columnspec%` is gone: one letter per column,
  in the table's order, with `|` between them where the vault draws column lines. Which columns
  wrap is your decision: the ones with the longest text. Name them in the closing line. A
  `<br>` in such a table becomes a paragraph break, which an `X` column breaks as intended.
- **A landscape page stands on its own.** The text before it ends its page, which can leave a
  gap; say so in one sentence.
- **A landscape page gives about half as much width again, not more.** A table wider than
  that still runs into the margin, without an error (measured: the probe table still 1.9 cm too
  wide, `Overfull \hbox (52pt too wide)` in the log). If the table is far too wide, say so and
  offer wrapping instead.
- **Shrinking makes the text small**, the more columns the smaller. Say so in one sentence,
  and that wrapping keeps the size.
- **A smaller font alone** (`\small`) was measured too and is not offered: in the probe table
  it was not enough.
- All three packages are in the templates' preamble; check the user's own preamble (see
  "Packages").

Measured 23.10.2026 through the converter and pdflatex with a four-column table that ran
300pt (10.6 cm) off the page: wrapping and shrinking fit exactly, landscape left it 52pt too
wide, `\small` 254pt; 0 errors in all of them. The log line `Overfull \hbox` is the only trace
of a table that runs off.

## Traps

- **A footnote inside a `tabular` cell is swallowed by LaTeX itself.** Not an Obsitex bug —
  put the remark before or after the table instead. `longtable` does not have this problem.
- **Very wide tables overflow the text block** without a warning. The three ways out are in
  "A table too wide for the page".
- **Inline backticks cut both ways.** Their content goes to LaTeX unescaped. That is what
  makes `` `\rowcolor{gray!25}` `` work — and it is why `` `snake_case` `` or `` `a^b` ``
  in a cell breaks the build or silently turns into maths. Use backticks in a cell **only**
  for a LaTeX command you mean. For literal text that merely looks technical, write it
  without backticks and let the converter escape it.

## Writing tables for a user

Keep the source readable: pad the columns so the pipes line up. Obsidian and the converter
do not care, but the author edits this file for months.

### Where the table goes: the marker line

**Your AI tool (Claude Code, Codex or OpenCode) does not see the cursor in Obsidian by itself.** The two programs share nothing
but the files on disk. So a table with no stated position lands wherever you judge best, and a
correct table in the wrong section still reads as a failure to the author.

The standard way, and the one the Blueprints tool hands the user, is a **marker line**: the
author types their marker (`<<<` unless they changed it; the request names it) on a line of its
own at the spot they want. Write the table in place of that line and delete the line. Until
23.10.2026 the marker line read `TABLE HERE`; a request that still says so means the same.

**Search the whole vault for the marker. Do not ask which file it is in.** The cards
deliberately do not carry a file name: the author knows where they typed the marker, not
what the note is called on disk ("03 Methods.md" when they think "the methods part"). A
grep over every `.md` in the vault costs nothing, measured 02.09.2026 at under a second
for 160 notes on OneDrive.

- **No marker anywhere? Then ask**, for the file and the spot in one question. Never fall
  back to the end of a file, and never pick a spot out of the surrounding prose on your own.
  One short question costs less than a table that has to be moved. If Obsidian is running,
  read its cursor first and offer that spot in the question (`cursor.md`).
- **More than one marker?** Ask which one, and name the file and the heading each sits under
  so the question can be answered without scrolling.
- The marker is plain text. Obsidian shows it as text, Obsitex would convert it as text, and
  nothing else in the chain reacts to it.
- **The request says "where my cursor stands in Obsidian" instead?** Then there is no marker
  line. Read the cursor (`cursor.md`) and write the table there, without asking first.

The second way is a described spot: "after the heading Results", "after the sentence about
the sample size". It arrives when the author answers the question above, so it needs no
separate handling. Confirm in one line where you put the table.

### Which table you mean: the cell marker

A file name does not identify a table. One note can hold three of them, and "the table in
03 Methods.md" then names a file, not a table. So the cards that change an **existing**
table ask for a marker inside any **cell** of the table meant. In a cell it breaks nothing:
the pipes and the column count stay as they are and Obsidian keeps rendering the table.

**The characters are the user's setting**, suggested as `<<<` but changeable, and the prompt
always names them. Read them out of the request. The prompt says "the table marked with
`<<<`" and does **not** say which cell, so search for the characters and take the table
around them. The full rule, including where the marker
sits for other elements and what to do when it is missing or doubled, is in
`obsitex-conventions.md`, "Which element the user means: the marker". Two things from there
matter in every table request:

- **Delete the marker in the same edit**, even when the request itself fails.
- **No marker? Ask**, and name the tables you found in the file under discussion by their
  heading and their first column titles.

### Building a new one: three ways in

The Blueprints tool offers three cards for a new table. They produce the same kind of table
and differ only in **how much you may ask**. Read the user's opening move and match it:

| The user arrives with … | You … | Card |
|---|---|---|
| the shape: columns, rows, headings, and says not to ask | write it, ask nothing | "Make an empty table" (`table-empty`) |
| nothing but the intention, and wants to be walked through it | ask the fixed questions, then write | "Build a table step by step" (`table-guided`) |
| pasted content | work out the form yourself, ask only where the data is ambiguous | "Make a table from my content" (`table-from-content`) |

**A user who did not come through a card gets way three**, unless they ask for questions.
Deciding and saying what you decided is cheaper for them than a questionnaire they did not
want.

#### How the rules below are sorted

Every decision about a new table is one of three kinds. Each kind is handled one way only.

| Kind | Meaning | What you do |
|---|---|---|
| **Fixed** | There is one right answer. | Do exactly that, in every way, whether you ask questions or not. |
| **Gap** | The spot is known, the value comes from a named source. | Take the value from that source and from nowhere else. No source, no value: leave it empty and say so in the closing line. |
| **Free** | Your judgement. | Decide, then name the decision in the closing line. |

**What is not listed below is not yours to change.** A wish for anything else (a grid, a
colour, a different placement) comes from the author, never from you.

#### Fixed, in all three ways

- **Position:** the marker line, see "Where the table goes" above.
- **Never invent a column heading.** They are the author's own words and they end up in a
  thesis: "Sample", "n", "p value" is a different table from "Group", "Count", "Share". A
  guessed heading looks finished, which is exactly why it survives to the printed version.
- **Never invent a caption, and this holds even where you ask nothing at all.** A heading
  labels something that is lying in front of you. A caption is a statement about the author's
  own work, and nothing in the data says what that work is. That is why a made up caption comes
  out long and explanatory: it had no source, so it was built out of the surroundings. **No
  caption is a valid result.** The converter simply leaves the `\caption{}` out (see "Captions"
  above) and nothing breaks. An empty spot the author can fill is honest. A sentence you made up
  is not, and it is the one that gets printed in the List of Tables.
- **Never invent a cell value.** An empty frame still needs at least one body row, so the
  author has somewhere to type and can see the column widths. Fill it with nothing, not with
  sample values: a table with invented numbers in it reads as data, and invented data in a
  thesis is the one mistake worth being paranoid about.
- **Keep the author's wording character for character**, typos included. If something looks
  like a typo, leave it and ask in the closing line.
- **Grid:** whatever `tableGridHorizontal` / `tableGridVertical` are set to in this vault.
  **Where the vault draws three lines** (its `tableEnvironmentText` carries `\toprule`), the new
  table gets the mark `` `\midrule` `` at the start of its first data row, see "Three lines".
  Without it the line under the heading row is missing, and nothing warns.
- **Placement:** leave `tableEnvironmentText` alone. The one exception is a table that runs over
  more than one page: then use a template from "Long tables" above, and only that.
- **Header row:** the first row, as a normal row. The Markdown table has no separate header
  markup. Shading is a separate wish, see "Colour" above.
- **Never ask whether the table should float or sit fixed.** The author does not know what a
  float is, and a question they cannot answer is worse than a default. It also comes too early:
  whether the placement bothers them shows up in the PDF, not while writing. If it does, the
  answer is in "Placement: why a table moves" above, and it starts by explaining the mechanism.
- **Closing line:** after writing, name your free decisions in one line, plus the one sentence
  each way asks for below.

#### Way one: they told you the shape

**Questions: none.** Not even about the caption.

| Kind | Decision | Source or default |
|---|---|---|
| Gap | number of columns, number of rows, column headings | the prompt. A heading missing where one is obviously needed: leave the cell empty |
| Gap | caption | none: a caption cannot be read off an empty table |
| Free | alignment per column | numbers and dates right, everything else left. The cells are empty, so judge from the heading |
| Free | whether the row count includes the header row | judge from the prompt |

**Say in the closing line** that the table has no caption yet, so it does not appear in the
List of Tables, and one word from them adds it. Do not leave this silent. The author sees a
finished table and has no reason to suspect that something is missing from a list they will
not look at for weeks.

#### Way two: they asked to be walked through it

This is the one place where a questionnaire is right, because the user chose it.

**Questions: exactly these, in this order, and no others.** Give an **example answer** with
each and say the example is a form, not a proposal.

1. what the columns are called, and roughly how many rows
2. whether it needs a caption and a number, and what the caption says
3. whether it may run over more than one page
4. whether the first row should be shaded

**Question 3 is a question the author can answer.** It is not the float question in disguise:
it asks about the amount of data, which they know, not about page breaking, which they do not.

| Kind | Decision | Source or default |
|---|---|---|
| Gap | column headings, number of rows | answer 1 |
| Gap | caption | answer 2, as given. If it is a full sentence, keep it and suggest a shorter noun phrase in the closing line. Never shorten it silently |
| Gap | more than one page | answer 3. A yes means one of the two templates in "Long tables" above, chosen by answer 2 |
| Gap | shaded first row | answer 4, done as in "Colour" above |
| Free | alignment per column | numbers and dates right, everything else left |

#### Way three: they pasted content

**Questions: only where the data itself is ambiguous.** A column that could be a heading or a
value, two candidate header rows, a unit that could belong to the heading or to every cell.

| Kind | Decision | Source or default |
|---|---|---|
| Gap | column headings and every cell | the material, character for character |
| Gap | caption | the material, character for character: a heading above the data, or a first line that reads as a title. If it names nothing, no caption |
| Free | alignment per column | numbers and dates right, everything else left |
| Free | more than one page | judge from the number of rows. If yes: a template from "Long tables" above, chosen by whether there is a caption |

**The caption is not one of your decisions.** If the material names nothing, **say so in the
closing line**: none was in the material, so the table stays out of the List of Tables and
cannot be cross-referenced, and you will add one the moment they name it. Reading that sentence
and answering it costs the author seconds. An invented caption costs far more, see "Fixed"
above.

#### What a caption looks like, wherever one comes about

**Short: a noun phrase, not a sentence.** "Design of the study", not "This table shows how the
study was designed". It stands in the List of Tables between other short lines, and it is what
a cross-reference points at. This describes the goal, it is never a licence to rewrite the
author's words: where their caption is a sentence, suggest, do not change.
