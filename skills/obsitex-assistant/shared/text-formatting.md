# Text and formatting

Emphasis, line breaks, maths, and the backtick trap.

> The general rules are in `obsitex-conventions.md` — including the paragraph rule, which is
> the most important thing on this page. This file gives the details.

## Emphasis

| Written | LaTeX | Result |
|---|---|---|
| `**bold**` or `__bold__` | `\textbf{…}` | bold |
| `*italic*` or `_italic_` | `\textit{…}` | italic |
| `***bold italic***` | `\textbf{\textit{…}}` | bold italic |
| `<u>underlined</u>` | `\underline{…}` | underlined |
| `==highlight==` | `\hl{…}` (soul) | highlighted |
| `~~strikethrough~~` | `\sout{…}` (ulem) | struck through |

`soul` and `ulem` are in the standard preamble — **no new package**.

`<u>` is the **only** HTML tag that works in body text; Markdown has no underline, and
Obsidian uses the same tag. Everything else lands raw in the PDF — see `not-supported.md`.

**Underscore emphasis only works at word boundaries.** `snake_case` stays literal, which is
what you want. Asterisks have no such guard: `word*part*` does become italic.

**Not supported: nested emphasis** like `*italic **bold** italic*`. The tokenizer works
left to right with one rule per construct and cannot balance them. Obsidian renders it,
Obsitex breaks it. Split it into separate spans, or use a raw block.

**Also not supported: emphasis spanning a line break.** `_renderInline` works line by line,
so a `**…**` pair whose halves sit on two lines is not recognised — the asterisks are printed
as text. It compiles cleanly and Obsidian renders it correctly, so **nothing catches this
but reading the PDF.** One more reason for the one-line-per-paragraph rule.

## Line breaks

A **single** newline inside a paragraph becomes a hard break (`\\`) — this matches Obsidian's
"strict line breaks: off". A **blank line** starts a new paragraph.

This is deliberate: a line break in the source is *content*, not formatting. It is also the
reason paragraphs must be written as one line. See `obsitex-conventions.md`.

## Maths

`$…$` inline and `$$…$$` display are passed through to LaTeX untouched — Obsidian maths
*is* LaTeX.

**Prices do not break formulas.** A `$` pair only opens a formula when all three hold:

1. no space after the opening `$`
2. no space before the closing `$`
3. no digit after the closing `$`

A formula also ends at the end of the line at the latest. "costs $5, plus $10" survives as
text because of rule 3. **Obsitex behaves exactly like Obsidian here** — what the author sees
as a formula in the preview becomes one in the PDF. To be certain, write `\$`.

## Turning text into a formula, by request

The card "Turn text into a formula" marks a passage that holds a formula, a unit or a symbol:
typed as plain text (`H2O`, `m2`, `10^6`), pasted from Word (`𝑦 = 𝑎 + 𝑏𝑥`, `α ≤ 0.05`) or
described in words ("the fourth root of x plus one"). Write it so it shows right in Obsidian
**and** in the PDF. Measured 25.10.2026, all of these in one PDF without an error:

| What it is | Write | Why |
|---|---|---|
| Chemical formula | `$\mathrm{H_2O}$`, `$\mathrm{CO_2}$` | `\mathrm` keeps the letters upright; plain `$H_2O$` sets them slanted like variables |
| Unit | `m$^2$`, `m$^3$` | only the exponent is a formula, the unit stays text |
| Power of ten | `10$^6$`, `10$^{-3}$`, `2.5 $\times$ 10$^{6}$` | more than one character after `^` or `_` needs braces |
| Variable with an index | `$x_1$`, `$x_n$` | slanted is right for variables |
| Greek letter, sign | `$\alpha$`, `$\le$`, `$\sqrt{2}$`, `$\sqrt[4]{x+1}$` | typed, they go missing in the PDF (`not-supported.md`) |
| Formula pasted from Word | rewrite it with plain letters, `$y = a + bx$` | Word's slanted letters (`𝑦`, U+1D466) go missing |
| A whole equation standing alone | `$$\bar{x} = \frac{1}{n} \sum_{i=1}^{n} x_i$$`, a blank line before and after | centred on a line of its own |

- **Only the formula goes between the `$`.** The words around it stay text. In a sentence it
  stays in the sentence (`$…$`); standing alone as its own paragraph it gets `$$…$$`.
- **Described in words: the formula replaces the words.** "the growth follows the fourth root of
  the sum of x and one" becomes "the growth follows `$\sqrt[4]{x+1}$`"; the words "the fourth
  root of the sum of" go. Found on the VM 25.10.2026 (TZ04): the words stayed and only `x + 1`
  became a formula, while the report claimed a root. Then say in words what the formula now
  says, so the user can check it, and make sure it matches what is in the note. If the
  description allows two readings ("the fourth root of x plus one": of x + 1, or of x and then
  plus 1), ask before you write.
- `±`, `×`, `°`, `²`, `³` typed as text come out right; inside a formula write `\pm`,
  `\times`, `^{\circ}`.
- The `$` rules above apply: no space after the opening `$` or before the closing one.
- A formula on its own line gets **no number** like "(1)", and a link cannot point at it.

Obsidian shows the formula set when the cursor is outside it, and the LaTeX while the cursor is
in it. The backtick form `` H`\textsubscript{2}`O `` also comes out right in the PDF, but Obsidian
shows it as code; use it only where a formula is not wanted.

## The backtick trap — read this before using backticks

**Inline backticks are NOT monospace text. Their content is passed to LaTeX raw.**

That is a feature when you mean it: `` `\newpage` `` or `` `\rowcolor{gray!25}` `` in a table
cell reach the output as commands. It is a defect when you do not:

- `` `snake_case` `` → `_` starts a subscript, or breaks the build
- `` `a^b` `` → `^` starts a superscript
- `` `#` ``, `` `$` ``, `` `%` ``, `` `&` ``, `` `{` `` → LaTeX commands, not characters

**To show a special character literally, leave the backticks off.** The converter escapes it
correctly on its own. Use backticks only for a LaTeX command you actually mean.

There is no monospace/verbatim inline form. If the user wants code-looking text, either
accept the plain escaped version or write `` `\texttt{…}` `` deliberately.

## Traps

- **Emphasis over a line break is silent** (see above) — the single most invisible defect in
  this file, because it compiles and Obsidian shows it right.
- **Emoji are kept, but they cost you twice.** They go into the `.tex` unchanged and Obsitex
  names every note that holds one. Still, `✅` or `❌` produce `Unicode character not set up for
  use with LaTeX`, one error per occurrence, and the character is missing from the PDF. On top
  of that an editor may refuse to edit a file that holds an emoji, Overleaf among them.
  Typographic characters are **not** affected: `→`, `—`, `…`, `©`, `®`, `™` and `°` compile
  fine. Best avoided in body text and headings.
