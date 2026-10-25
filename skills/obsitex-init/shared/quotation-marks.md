# Quotation marks

> The general rules are in `obsitex-conventions.md`. This file explains the counting
> mechanism, which causes more surprises than any other conversion.

## How the conversion works — the whole logic

Obsitex converts **only the straight double `"`**, and it does not track whether a quote is
open. It simply counts:

> **Odd `"` → opening. Even `"` → closing.**

```
He said "yes" and she said "no".
        1   2             3    4
        ↓   ↓             ↓    ↓
        «   »             «    »
```

Which characters are inserted comes from the DDS keys `openingQuotationMark` /
`closingQuotationMark`. **The count runs across the whole document** — a quote left open in
chapter 2 keeps acting in chapter 3.

**One stray `"` inverts everything after it.** All following pairs come out backwards, and it
usually only shows up in the finished PDF.

## Three ways to write quotes

**1 — Type straight `"`.** Convenient; the converter does the rest. Fine for plain
sequential quotes.

**2 — Type the real characters yourself** (`«…»`, `„…"`). They pass through untouched, so no
stray mark elsewhere can invert them. That immunity is also the price: they are outside the
DDS setting for good (see "Pick one and stay with it"). Required for a straight `"` that has
to stay straight in the PDF, such as inches or code.

**3 — `\enquote{…}` in backticks.** `csquotes` is in the standard preamble and knows the
levels itself, so nesting is automatic and a missing brace is a build error rather than a
silent inversion. **But:** special characters inside are not escaped, so a `%` or `&` in the
quote breaks the build; the characters come from the babel language, not from the DDS keys;
and another backtick in the same paragraph pairs wrongly. Powerful but brittle.

### Pick one and stay with it — for the whole document

The three ways are alternatives, not a menu to choose from per sentence. **Mixing them is the
one thing that actually breaks**, because the count runs across the whole document: a straight
`"` dropped into a passage written with typed characters inverts every pair after it, to the
end of the thesis (see "Traps").

Way 1 is the default and the one to recommend, for one reason that only shows up later: **only
a straight `"` is still under the DDS setting.** Characters typed by hand are finished text.
Switch the document from `«»` to `„"` a year in, and every hand-typed spot stays as it was —
no warning, no list of them.

**When you write into the user's manuscript, this is a hard rule and not a preference**
(`obsitex-assistant/SKILL.md`, "Hard rules"): type the straight `"`, unless the file already
carries typed characters, in which case match what is there.

## Quotation styles, by request

The card "Change the quotation marks" names one of six styles. Each is quoted here word for
word, with the two keys of the `dds` block in `00 Document Setup.md` (measured 25.10.2026, all
six in one PDF without an error):

| The prompt wants … quotation marks | `openingQuotationMark` | `closingQuotationMark` | In the PDF |
|---|---|---|---|
| "English “…”" | `“` | `”` | “text” (the template's standard) |
| "British ‘…’" | `‘` | `’` | ‘text’ |
| "German „…“" | `„` | `“` | „text“ (what `obsitex-init` sets for German) |
| "German book-style »…«" | `»` | `«` | »text« |
| "Swiss «…»" | `«` | `»` | «text» |
| "French « … »" | `«\\,` | `\\,»` | « text », with a narrow fixed space inside |

**French needs the space in the value**, written as `\\,` in the JSON (LaTeX `\,`, a thin space
that never breaks). Measured: a plain space `"« "` lets the line break between the mark and the
word; `"«~"` is a full fixed space, too wide.

Change **only these two keys**. Where depends on the request:

- **"in the whole PDF":** in `00 Document Setup.md`. A `dds` block lower down changes them again
  from there on; leave such blocks alone and name them.
- **"in the part I marked"** (since 25.10.2026, for example an English abstract in a German
  thesis): a ` ```dds ` block with the two keys where the part begins, and a second one where it
  ends that puts back the two values in force at the beginning (the last `dds` block above it
  that sets them, else `00 Document Setup.md`), character for character. With no second marker,
  the first block alone. Leave `00 Document Setup.md` as it is. How a part is found and where
  its blocks stand: `obsitex-conventions.md`, "A part is not the document setup".

**A part changes which marks print, not which `"` opens.** The count runs on across the part,
so the pairs inside it must be complete as everywhere. Measured 25.10.2026 through the converter
and pdflatex: a German document („…“), a part across two notes set to English, “why” and “done”
inside, „ja“ before and „wieso“ after, 0 errors.

**Then list the marks typed by hand or pasted** in the manuscript, or for a part only inside it (search for `“ ” „ « » ‘ ’ ‹ ›`).
They do not follow the setting. Leave out an apostrophe, a `’` between two letters (`don’t`).
Ask before you change them. On a yes, turn an outer pair into straight `"…"`, so it follows the
setting from now on (one pair for one pair keeps the count even), and an inner pair into the
inner marks of the new style (the table in the next section; British inner is `“…”`, German
book style inner `›…‹`, for French ask).

## Finding a stray mark (point 9 of the check before converting)

A `"` left over compiles **without an error** and turns every quotation after it the wrong way
round, in the following notes too, to the end of the document (measured 25.10.2026: after one
missing mark, “yes” printed as ”yes“, and the next chapter's “efficient” as ”efficient“).

- **Count the straight `"` in the body text of each note.** An odd number means one is left
  over. A `"` inside backticks or a code block does not count (measured: the quotations after
  `` `\textbf{"}` `` stayed right).
- **Find the spot:** the first quotation that would come out the wrong way round, or a `"` that
  opens and is never closed in its paragraph. Name it with its line, and the nearest fix: add the
  missing mark, or remove the stray one.
- **Also list marks typed by hand or pasted** (`“ ” „ « » ‘ ’`, not an apostrophe between two
  letters). They do not follow the setting; offer to turn them into straight `"`.
- The check changes nothing. Fixing is the next request.

## Nested quotes — the mixed form

**Obsitex cannot detect nesting**, because a straight `"` looks identical at both ends and
carries no level information. `"outer "inner" more"` counts the third mark as a closer.

The missing information can only come from the author. **Write the inner marks yourself and
leave the outer pair as `"`** — characters you type do not affect the count:

```markdown
"The term ‹internal control system› is used narrowly here."
```

| Level | Switzerland | Germany | English |
|---|---|---|---|
| outer (type `"`) | `«` `»` | `„` `"` | `"` `"` |
| **inner (type yourself)** | `‹` `›` | `‚` `'` | `'` `'` |

This is the most comfortable way to nest: only two characters by hand.

## The apostrophe

Obsitex converts **only** the double `"`. The single `'` always passes through — deliberately,
because in English it is the apostrophe. `don't`, `it's`, `Anna's` come out correctly;
LaTeX makes a typographic apostrophe by itself. Intervening here would break every second
English sentence.

**But `'` is useless as an inner quote:** LaTeX always sets it as a *closing* mark, so
`'inner'` becomes `'inner'` with the opening one facing the wrong way. Type the real
characters instead.

**The LaTeX way of writing an opening single quote is a backtick — and that does not work in
Obsidian**, where a backtick starts inline code. If real code sits in the same paragraph, the
backticks pair wrongly and the text between them vanishes as raw LaTeX.

## Traps

- **Autocorrect is the most common cause of mixed quotes in one document.** Obsitex reacts
  only to the straight `"`. If the editor or the operating system has already converted it to
  `"` or `„`, those characters pass through unchanged — so part of the document has `«»` and
  part has `""`, with no warning. **Obsidian itself replaces nothing**; the culprit is almost
  always **text pasted from Word**, LibreOffice, macOS or iOS.
  *Check:* search the Markdown for `"` and `„`. Anything found did not come from the keyboard.
- **Counting is document-wide.** When you add or remove a quotation mark anywhere, everything
  after it can flip. Say so if you edit text containing quotes.
- **A `dds` change of the quote characters is not retroactive** — quotes above keep the old
  marks.
