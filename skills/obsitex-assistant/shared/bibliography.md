# Bibliography and citations

Embedding a `.bib` file, adding sources to it and citing from it: a source on its own, short and
long quotations, paraphrases.

> The general rules are in `obsitex-conventions.md`. This file gives the citation-specific
> cases.

## Placing the bibliography

Embed the `.bib` file where the list should appear, usually at the end of the document:

```markdown
![[refs.bib]]
```

One embed does **two** things: `\addbibresource{…}` goes into the preamble, and
`\printbibliography` is emitted at the embed's position — from the DDS fields
`bibliographyCmdText` and `bibliographyBodyCmd`. On export the `.bib` lands in a `bib/`
subfolder.

The standard preamble already loads `\usepackage[backend=biber, style=numeric-comp]{biblatex}`
— **no new package**. How a citation looks (`[1]` vs. "Müller 2024") is decided by the `style=`
option there, **not** by Obsitex.

**Changing the style needs a second line.** The converter always emits `\cite{…}`, and `\cite`
means something different in every biblatex style. The second line, `\let\cite\autocite`, goes
right below the `biblatex` line; `\autocite` is what each style means by a citation:

| Style | Extra line in the preamble | In the PDF | Without the line |
|---|---|---|---|
| `numeric-comp` | none | `[1]`, `[1, p. 25]` | (the same) |
| `authoryear` | `\let\cite\autocite` | `(Knuth 1984)`, `(Knuth 1984, p. 25)` | `Knuth 1984`, **no parentheses** |
| `apa` | `\let\cite\autocite` | `(Knuth, 1984)`, `(Knuth, 1984, p. 25)` | `Knuth, 1984`, **no parentheses** |
| `verbose` | `\let\cite\autocite` | a footnote **after** the full stop: the whole reference the first time, `Knuth, The TeXbook, p. 25.` after that | the **whole reference inside the sentence** |

Measured 14.08.2026 (`apa`: 07.10.2026), the extra line 25.10.2026 with all three. Until then
the skills wrote `\let\cite\parencite` (the same result) and `\let\cite\footcite`, which put the
footnote mark **before** the full stop ("written¹."); `\autocite` puts it after ("written.¹").
All broken forms **compile without a warning**, they are only visible in the PDF. Back to
numbers: `style=numeric-comp` and the extra line goes. The list of references follows the style
by itself.

**`numeric-comp` numbers the sources in the order of the authors' names, not in the order they
are first cited** (measured 25.10.2026: Meier, Keller and Knuth cited in that order print [3], [1],
[2]). Only `ieee` numbers in the order of first citation. Say so when a user switches to numbers.

### Citation styles, by request

The card "Change how citations look" names one of four styles in a sentence. Each is quoted
here word for word, with what it means:

| The prompt wants citations … | `style=` | Extra line |
|---|---|---|
| "as numbers in square brackets, like [1]" | `numeric-comp` | none (remove one that is there) |
| "as author and year in parentheses, like (Knuth 1984)" | `authoryear` | `\let\cite\autocite` |
| "in APA style, like (Knuth, 1984), with the list of references in APA style too" | `apa` | `\let\cite\autocite` |
| "as footnotes, with the full reference the first time and a short one after that" | `verbose` | `\let\cite\autocite` |

Change only the `biblatex` line and the extra line, in the `latex-preamble` block of the document
setup. Then say what else changes: the list of references takes the new form; under APA check the
titles (below); with footnotes, the citations join the footnotes in their numbering
(`footnotes.md`), and a paraphrase prenote stands at the start of the footnote ("cf. Knuth, …").

## APA: titles in the `.bib`

APA writes the titles of articles and books in sentence case, and `style=apa` enforces it:
**every word after the first is lowercased** in the reference list. The other styles leave
titles alone. Two cases go wrong (measured 07.10.2026):

| Case | `.bib` | In the PDF | Fix |
|---|---|---|---|
| Proper noun | `title = {Writing in Obsidian at the University of Zurich}` | "Writing in obsidian at the university of zurich" | braces around it: `{Obsidian}`, `{University of Zurich}` |
| Source in another language | a German title, English document | "Ein beispielartikel für den test-vault" | `langid = {ngerman}` in that entry |

Braces protect only what they enclose, so put them around the name, not the whole title.
When a user picks APA or adds a source under APA, check the titles for both cases.

## The four citation forms, by request

The card "Cite a source" marks a passage with two marks (or the user selects it) and says what
the passage is, in one of four sentences. Each sentence is quoted here word for word, with what
to write. The source goes **after** the passage, inside its sentence, before the full stop.

| The prompt says the passage … | Write |
|---|---|
| "needs its source after it" | `The market is efficient [@mueller2024].` With a page: `[@mueller2024, 14]` |
| "is a word-for-word quotation: put it in quotation marks, with its source after it" | `As Müller writes, "the market is efficient" [@mueller2024, 14].` |
| "is a long word-for-word quotation: set it apart as an indented block without quotation marks, with its source at the end" | a `>` block, the source at the end of its last line (below) |
| "is someone else's idea in my own words: mark it as a paraphrase, with its source after it" | `The market is generally held to be efficient [cf. @mueller2024, 14].` |

Measured 24.10.2026, all four in one PDF without an error: "[1]", "[1, p. 14]", an indented
block ending in "[1, p. 15]", "[cf. 1, p. 14]" (the template's numeric style).

**The page is the bare number:** `14`, or `25-27` for a range. biblatex writes the word itself,
in the language of the document: "p. 14" and "pp. 25–27" in English, "S. 14" in German
(measured 25.10.2026). **Do not write `p.~14`.** It works in English, but a German thesis then
prints "[1, p. 14]". "No page" in the prompt means no locator at all: `[@mueller2024]`.

**Short quotation.** Straight `"…"` around the passage; the converter turns them into
typographic quotes (`quotation-marks.md`). If the passage already stands in quotation marks,
keep them and do not add a second pair.

**Long quotation.** Every line of the block starts with `>`, the source closes its last line:

```markdown
> A longer quotation running over
> several lines.
>
> Several paragraphs are possible. [@mueller2024, 15]
```

It becomes an indented `quote` environment **without** quotation marks, as is usual in academic
writing; remove any that stood around the passage. A blank line inside the quote is a `>` alone
on its line. The block needs a blank line before and after it.

**Paraphrase.** No quotation marks. The prenote is the word for "compare" in the language of the
document, **inside** the brackets: `cf.` in English, `vgl.` in German. `[cf. @mueller2024, 14]`
→ `\cite[cf.][14]{mueller2024}`. The text **before** the `@` becomes the prenote, and **there
must be a space between the prenote and the `@`**.

**Which source.** The prompt names it loosely ("Müller 2024"), pastes the full reference or
gives a DOI. Read the `.bib` the vault embeds and find the entry by author and year, by title or
by DOI. Use its key. Not there: with a full reference or a DOI, add the entry first ("Adding
sources" below, a DOI is looked up) and say so; with only author and year, ask for the full
reference. **Never invent a key or an entry.**

## Adding sources

The card "Add sources to the bibliography" pastes one or more references, in any form: APA, a
library entry, a BibTeX entry, a reading list, or only DOIs. It only stores them. **Cite nothing**; the
card "Cite a source" does that.

- Write each source as an entry into the `.bib` the vault embeds (`![[refs.bib]]`), with the
  type that fits (`@book`, `@article`, `@incollection`, `@online`, …).
- **Key:** in the pattern of the keys already there; otherwise surname and year in lower case
  (`mueller2024`). A second work by the same author in the same year gets `a`, `b`.
- **Already there** (same title, or same DOI): say so and keep the existing entry and key.
- **Only what the reference says.** A field it does not give stays out. Never add a page range,
  a publisher or a DOI from memory. **A DOI is the exception** (next point).
- **A DOI: look it up.** When a reference carries a DOI (`10.…`, also as `https://doi.org/…`),
  or is only a DOI, ask doi.org for the entry. The data then comes from the agency that issued
  the DOI, not from memory (measured 25.10.2026):

  ```
  curl -sL -H "Accept: application/x-bibtex" https://doi.org/10.1038/nature14539
  Invoke-RestMethod -Uri https://doi.org/10.1038/nature14539 -Headers @{Accept = "application/x-bibtex"}
  ```

  The answer is a BibTeX entry. Take its fields, then fit it to the vault: the key in the
  vault's pattern (`LeCun_2015` → `lecun2015`), page ranges with two hyphens (`436–444` →
  `436--444`). **Leave out three fields** the list of references would print (measured
  25.10.2026 with a PLOS article): a `url` that only repeats the DOI ("url: http://dx.doi.org/…"
  beside "doi: …"), the `ISSN` ("issn: 1932-6203"), and in an `@article` the `editor`, which is
  the journal's handling editor and prints as "Ed. by Raymond Niaura". Where the pasted text and
  the answer differ, keep the answer and name the difference. A DOI tool from an MCP server, if the
  user has one, does the same and may be used instead.
- **The lookup fails** (no internet, or doi.org answers "Not Found"): say so. Use only what the
  pasted text gives, keep the DOI as given and tell the user to check it. Was it only a DOI, ask
  for the full reference.
- Under APA, check the two cases in "APA: titles in the `.bib`" above.
- Tell the user, for each source, its key and that it is cited as `[@key]`, or with the card
  "Cite a source" by author and year.

## Traps

- **Citation keys must exist in the `.bib`.** A key that is not there prints as a question
  mark in the PDF. **Never invent keys** — read `refs.bib` first, or ask the user.
- **Several sources in one bracket (`[@a; @b]`) are not supported.** Way out: raw LaTeX in
  backticks — `` `\cite{a,b}` ``.
- **A bracket containing `@` without a space before it is not a citation** — `[name@firm.ch]`
  stays ordinary text. Useful to know, but also a reason to write email addresses outside
  brackets.
- **The bibliography needs two LaTeX passes plus biber.** If the user reports empty brackets
  or missing entries after one run, that is the cause, not the Markdown.
- **A caption on the `.bib` embed is ignored** — deliberately. `![[refs.bib|Sources]]` does
  nothing with the text.
