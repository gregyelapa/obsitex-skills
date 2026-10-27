# Links and cross-references

> The general rules — in particular **how to write an attachment link** — are in
> `obsitex-conventions.md`. This file covers linking to headings and external URLs.

## The core idea

**A wikilink points at a heading — a `#` line.** In the PDF it becomes a clickable
cross-reference ("Section 3.2"). The file name in the link **never appears** in the PDF; it
only says which file the heading is in. **A table caption and an image caption work the same
way** after the `#`; that is the "Traps" section below and `tables.md` / `images.md`.

| Written | Points at |
|---|---|
| `[[Note#Heading]]` | that heading in "Note" |
| `[[#Heading]]` | a heading in the **same** file |
| `[[Note]]` | the **first** heading of "Note" |
| `[[Note#Heading\|my text]]` | as above, but shows "my text" |

Files are addressed by **file name** — exactly the one Obsidian suggests while typing `[[`.
A number prefix may stay in (`[[01 Methods#Design]]`); it does not appear in the PDF.

Resolution is **path/file-name based only** (relPath · relPath without `.md` · basename,
lowercased). `title:` and `aliases:` do **not** count — there is no need to set them.

**Where the reference goes** comes with the request: the short marker in the sentence (put the
link in place of it, `obsitex-conventions.md`) or the cursor in Obsidian (`cursor.md`, step 3,
inserted as part of the sentence, with a leading space if the cursor follows a word). **What it
points to** is named in words, often loosely ("the chapter on methods", "the survey table").
Find the target in the vault and use the form this file gives for it: a heading directly, a
table or a figure by its caption. A table without a caption cannot be reached; say so and offer
to give it one, do not link to a heading near it instead.

## What it produces

`\vref{label}` via the DDS field `crossReferenceCmdText`; with an alias, `\hyperref`. The
label is built path-uniquely from file path **plus** heading, byte-identical at definition and
reference — so several files with a `# Introduction` do not collide.

### What the reader actually sees — four parts, three packages

A resolved reference prints something like this (measured 01.09.2026, capital since 24.10.2026):

```
Table 3 on the preceding page
^^^^^ ^ ^^^^^^^^^^^^^^^^^^^^
type  no      position
```

1. **The type word** comes from **`cleveref`**, not from `\vref`: "Table", "Figure", "Section".
   With a capital because the standard preamble gives `cleveref` the option `capitalise`; see
   "Small or capital" below.
2. **The number** — from the counter, and a counter is stepped by a **caption**. No caption,
   no number (see `tables.md`).
3. **The position** — from **`varioref`**. It chooses between "on the preceding page",
   "on page 12" and **nothing at all** when the target sits on the same page. So a reference
   that reads just "figure 2" is not broken; the figure is simply on that page.
4. **The jump** — from **`hyperref`**. Invisible, but clickable.

**`\vref` on its own gives only 2 and 3.** The type word appears because the standard preamble
loads `cleveref` **after** `varioref`, and cleveref rewrites `\vref`. Measured with and without:

| Preamble | `\vref{tab:t}` prints |
|---|---|
| `varioref` only | `1` |
| `varioref` + `cleveref` | `table 1` |
| `varioref` + `cleveref` with `capitalise` (the standard since 24.10.2026) | `Table 1` |
| the same with `ngerman` | `Tabelle 1` |

**Four things that matter for a thesis in another language.** All are easy to get wrong, and
apart from French none stops the PDF:

- **The language of the type word lives in the package options**, not in babel. Switching the
  document to German means `\usepackage[ngerman]{varioref}` **and**
  `\usepackage[noabbrev,capitalise,ngerman]{cleveref}`. Change babel alone and the text stays
  German while every reference keeps saying "Table".
- **The two packages do not always use babel's name for a language.** A name a package does
  not know is an error in the log, and the PDF still builds, with English words: "table 1 on the
  preceding page" in a Portuguese thesis. The names that work (measured 27.10.2026):

  | Document language | `babel` | `varioref` | `cleveref` | `capitalise` |
  |---|---|---|---|---|
  | English | `english` | `english` | `english` | keep |
  | German | `english, main=ngerman` | `ngerman` | `ngerman` | keep, it changes nothing |
  | French | `english, main=french` | `french` | `french` | take out |
  | Spanish | `english, main=spanish` | `spanish` | `spanish` | take out |
  | Italian | `english, main=italian` | `italian` | `italian` | take out |
  | Portuguese, Brazil | `english, main=brazilian` | `brazil` | `brazilian` | take out |
  | Portuguese, Portugal | `english, main=portuguese` | `portuges` | no language, and the names below | take out |

  `english` stays in the `babel` line for the reason `obsitex-init` gives for German (varioref
  under TeX Live 2026); it does not change the language of the document.

  `cleveref` has no European Portuguese, so it needs its words spelled out, at the very end of
  the `latex-preamble` block (`\crefname` only works in the preamble):

  ```latex
  \crefname{chapter}{capítulo}{capítulos}
  \crefname{section}{secção}{secções}
  \crefname{subsection}{secção}{secções}
  \crefname{subsubsection}{secção}{secções}
  \crefname{appendix}{apêndice}{apêndices}
  \crefname{subappendix}{apêndice}{apêndices}
  \crefname{table}{tabela}{tabelas}
  \crefname{figure}{figura}{figuras}
  \crefname{equation}{equação}{equações}
  ```

  `paragraph` and `subparagraph` already have their own `\crefname` lines in the block about
  the numbering depth; write them as `parágrafo` and `subparágrafo` there, not a second time.
  For another language: look up both packages' option names, and when one is missing, write
  the `\crefname` lines the same way. Say that this language is untested.

  **French builds no PDF at the moment** (measured 25.10.2026). Every label Obsitex writes holds
  a colon (`sec:…`, `fig:…`), French babel makes the colon an active character under pdflatex,
  and every cross-reference then stops with `Missing \endcsname inserted`. Before setting up or
  switching a document to French, say so plainly. Spanish and Italian are not measured for this.
- **`capitalise` follows the language** (last column). French, Spanish, Italian and Portuguese
  write these words small in a sentence ("le tableau 1", "la tabla 2"), and `cleveref` without
  `capitalise` prints exactly that. In German the nouns have a capital anyway. In English it is
  a matter of style; see "Small or capital" below.
- **`noabbrev` is not decoration.** Without it cleveref abbreviates: "tab. 1" instead of
  "table 1". The standard preamble sets it, in every language.

### Small or capital

A reference cannot know whether it starts a sentence. Obsitex writes the same `\vref` for every
link, and guessing a sentence start goes wrong ("vgl. [[…]]", "e.g. [[…]]"). So the case is one
setting for the whole document, in the `cleveref` line of the preamble (measured 24.10.2026):

| `cleveref` option | in a sentence | at the start of a sentence |
|---|---|---|
| with `capitalise` (standard in English since 24.10.2026) | as Table 1 shows | Table 1 shows |
| without `capitalise` | as table 1 shows | table 1 shows (wrong) |
| without `capitalise`, the paragraph inside the `\Vref` pair below | as Table 1 shows | Table 1 shows |

German prints "Tabelle 1" in every row, because its nouns have a capital anyway. French,
Spanish, Italian and Portuguese print small without `capitalise` too ("tableau 1"), and the
`\Vref` pair gives them a capital ("Tableau 1"), measured 27.10.2026.

In English both are right; it depends on the style guide. APA and IEEE write a capital in a
sentence ("as Table 1 shows"), Chicago and MLA a small letter ("as table 1 shows"). In the
languages above that write it small, `capitalise` is out from the start (table in the section
before).

**English vaults made before 24.10.2026 lack `capitalise`.** When a reference there starts a
sentence, say that the PDF prints it small and offer to add `capitalise` to the `cleveref` line.
Say in the same offer that it works for every reference in the document, so the ones in running
text get a capital too ("as Table 1 shows"). The user decides knowing that.

**The `\Vref` pair** gives the references of one paragraph a capital while `capitalise` is out: a
`dds` block before the paragraph with `{"crossReferenceCmdText": "\\Vref{%labelref%}"}` and one
after it that puts back what the document setup has, normally
`{"crossReferenceCmdText": "\\vref{%labelref%}"}`. (If the setup has `\\cref{%labelref%}`, the
first block takes `\\Cref{%labelref%}` and the second `\\cref{%labelref%}`.) **Every** reference
in that paragraph is then written with a capital, not only the one at the start of a sentence;
say so when there is another one. The note shows two code blocks per paragraph.

#### The card "Capital or small letter in cross-references"

It asks for one of these two, word for word, for the whole document:

| The request says | It means |
|---|---|
| with a capital letter everywhere, like "as Table 1 shows" | `capitalise` in the `cleveref` line |
| with a small letter inside a sentence, like "as table 1 shows", and a capital letter at the start of a sentence | no `capitalise`, and the `\Vref` pair around each paragraph with a reference at the start of a sentence |

Read the document language from the `babel` line first.

- **German** (`ngerman` is the main language): change nothing. Say that German prints "Tabelle 1"
  with a capital everywhere, because German nouns have one, so neither choice changes the PDF.
- **A capital everywhere:** add `capitalise` to the `cleveref` line if it is missing. Take out
  every `\Vref` pair; it is not needed any more. Say that this gives every reference a capital,
  in running text too. In French, Spanish, Italian or Portuguese say also that their own rule
  writes these words small in a sentence; the user decides.
- **Small inside a sentence:** take `capitalise` out of the `cleveref` line. Then look through
  every note of the manuscript for a reference at the start of a sentence: a `[[…]]` link at the
  start of a paragraph or of a list item, or after a full stop, a question mark or an exclamation
  mark that ends a sentence (not after "e.g." or "cf."). Put the `\Vref` pair around each
  paragraph that has one, unless it already has a pair; for a list item, around the whole list,
  because a code block between two items splits the list in two (measured 27.10.2026). Name
  each place. Say that a reference written later at the start of a sentence prints small
  until it gets a pair, and that "Check my manuscript before converting" names those.

In every case, end with what changed in plain words, and name a paragraph where another reference
now gets a capital as well.

#### Checking before converting

"Check my manuscript before converting" asks under point 2 for every link that starts a sentence
while the `cleveref` line has no `capitalise`. Leave out a German document and a paragraph inside
a `\Vref` pair. For each one, say that the PDF prints it with a small letter, and name the two
ways: the `\Vref` pair for that paragraph, or `capitalise` for the whole document.

**Do not write the command out** (`` `\Vref{subsec:…}` `` in backticks) to reach one link alone.
It works until the target moves: its name is built from the heading's level, the file and the
heading. A renamed heading or file, or a heading one level deeper, prints "?? on page ??", and
the PDF still builds without an error (measured 24.10.2026). A `[[…]]` link survives all of that.
If the user insists, write it and say this.

**When the user wants the reference without its page**: put
`"crossReferenceCmdText": "\\cref{%labelref%}"` into the `dds` block of the document setup. Every
reference then prints "Table 1" without "on the preceding page".

**An appendix needs two lines after `\appendix`** (measured 07.10.2026). Without them a
reference to an appendix says "chapter B" instead of "appendix B", in every class and every
language. `Switch to Appendix` carries them since version 1.62.2; older vaults lack them. The
names `appendix` and `subappendix` in them are internal; the printed word ("Anhang",
"annexe") comes from the `cleveref` language option like every other type word. Details and
what to offer the user: `headings.md`, "Switch to Appendix".

External links: `[text](https://…)` → `\href`; a bare URL in running text → `\url`, clickable.

## What does not link — and stays as readable text

None of these break anything; they simply print instead of linking:

- **A broken link** (typo, heading renamed) — the visible text stays.
- **A block reference to a paragraph** `[[Note#^id]]` — not a target. Only figures and tables
  carry a label of their own; a paragraph would inherit the section's number. The id itself is
  removed from the text either way, and the converter logs warning 92650. Figures and tables
  DO work this way, see below.
- **A note transclusion** `![[Note]]` — the content is **not** inserted; `![[Note]]` appears
  visibly as text so the author notices.
- **A file with no `#` heading at all** is not linkable. `[[Note]]` needs a first heading to
  aim at.

## `[[…]]` vs. `![[…]]` — do not mix them up

- `[[…]]` = **reference**, a jump to a heading.
- `![[…]]` = **embed**. For **files** (`![[image.png]]`, `![[appendix.pdf]]`,
  `![[refs.bib]]`) the content is embedded — see `images.md`, `pdf-embeds.md`,
  `bibliography.md`. For **notes** it does not work (see above).

## Traps

- **Headings, captioned tables and figures can be targets, nothing else.** A table or an image
  is reached by its caption after a `#`: `[[#Overview of the design]]` or
  `[[Note#Overview of the design]]` (tables since 29.08.2026, images since 01.09.2026; see
  `tables.md` and `images.md`). A table with no caption has no label and cannot be reached; an
  image without one still can, because the converter makes its file name the caption. **A bare
  `[[Overview of the design]]` is none of these** — that namespace belongs to note names, so
  the `#` is required. **Paragraphs still cannot be linked.**
- **A link to a caption looks dead inside Obsidian.** Obsidian resolves `#` only to headings,
  so it paints a caption link as unresolved and says "not found" on hover. Obsitex converts
  it correctly anyway. The cost is real though: no click while writing, and Obsidian does not
  follow a renamed caption. See `tables.md` and `images.md`.

## Block identifiers: the form that lives in Obsidian too

Obsidian names a single block with `^id` at the end of it, and writes that id **by itself** the
moment somebody picks a block from the `[[Note#^` autocomplete. Since 02.09.2026 Obsitex reads
it:

```markdown
![[figure.png|Sample distribution]]
^dist

Shown in [[#^dist]].
```

```latex
\begin{figure}[!htbp]
  \includegraphics[width=\textwidth]{./images/figure.png}
  \caption{Sample distribution}
  \label{fig:sample_distribution}
\end{figure}

Shown in \vref{fig:sample_distribution}.
```

**No second `\label`.** The figure already has one from its caption; the id is just a second
name for it. The output is identical to `[[#Sample distribution]]`.

**Three positions all work** (measured in Obsidian): on its own line after the block, on its own
line after a **blank** line, and appended to the block's last line. A blank line does not detach
the id from the block above it.

**When to suggest which:**

| | Caption anchor | Block identifier |
|---|---|---|
| Clickable in Obsidian while writing | no | **yes** |
| Survives a rewritten caption | **no**, breaks silently | yes |
| Readable in the markdown | **yes** | no, Obsidian invents `^a3f2b1` |

The caption is the normal way because it reads well. Reach for a block id when the author wants
to click their own references while writing, or when a caption is still in flux.

**Never on a paragraph.** See the list above: it is removed but is not a target.
- **Renaming a file outside Obsidian breaks every wikilink pointing at it.** Obsidian
  updates them, the file explorer does not. Rename in Obsidian, and answer "Always update".
  If it must be done outside, update the links **and** the Flexplorer `data.json` in the
  same operation.
- **Renaming a heading breaks links to it** the same way, and nothing warns — the link
  quietly becomes plain text in the PDF.
- **`[Text][id]` reference links are not supported**, and their definition line is printed as
  a paragraph. See `not-supported.md`.

## Moving a chapter

Moving a chapter changes its position, not its file. With the Flexplorer
plugin, change only the folder's `customOrder` in `data.json`: take the
entry out and put it back before the other chapter. Leave the files where
they are and keep their names. No link breaks, so there is nothing to
repair. Say so in the answer. Writing `data.json` needs no Obsidian command
line and no restart (assistant skill, "Entering a new note in the
Flexplorer order").

With number prefixes and no Flexplorer, the name carries the order, so
moving means renaming. Give the file a free number between its new
neighbours (`35` between `30` and `40`). Never renumber the other files.
If no number is free, ask. A rename breaks every link that names the old
file (`[[50 Results]]`), so update all of them in the same step.

If the other chapter sits in another folder, the file has to move there.
First check whether the move changes the level of its headings
(`headings.md`, "The folder note: what makes a folder a level"). If it
does, stop: say what the heading becomes and ask before you move anything.
Only then move it on disk yourself. Do not use the Obsidian command line,
and do not stop because it is off. Take the entry out of the old folder's
`customOrder` and put it into the new one. A link that names the note only
(`[[Methods]]`) still works; update every link that spells out the old
path.
