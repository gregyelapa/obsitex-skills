# PDF embeds

Including a whole PDF document — a scanned form, a questionnaire, a paper — in the thesis.

> The general rules, including how to write an attachment link, are in
> `obsitex-conventions.md`. This file only gives the PDF-specific cases.

## A PDF the user dropped into the chat

A PDF arrives differently from an image. A pasted image comes off the clipboard and has no
name, which is why `images.md` sends you there. A PDF hung into the chat arrives **with its
file name and its extracted text** — that is enough to find the original on disk. So look
for it there first, and do not ask the user for a path before having searched.

1. **Search the file name** across the places a downloaded document lands — `Downloads`,
   `Desktop`, the OneDrive folders, the vault itself.
2. **Verify the find by its content.** Read page 1, or compare it against the text the chat
   already extracted. A matching name is not proof: `Fragebogen.pdf` may exist three times in
   three versions. This step is what separates "a file of that name" from "the file meant".
3. **Copy it into the attachments folder.** Which folder that is, is decided exactly as for
   images — `images.md`, step 3 of "The procedure". The rule with `app.json` and an existing
   `attachments` folder holds unchanged; an attachment belongs *inside* the manuscript.
4. Keep the file name or give it a speaking one. Unlike an image it drives nothing: a PDF
   embed has no caption and no label (see "Traps").

Only when the search finds nothing: ask where the file is. Never before.

## After the file is saved: ask, do not decide

The file lying in the vault is half the job. **Where it goes in the document is the user's
decision, not yours** — and when the answer is not already in what they wrote, ask it as one
structured question. Do not hand the task back with instructions for doing it in Obsidian:
creating a note, including its place in the order, is something this skill does now
(`SKILL.md`, "Working in the user's vault").

Ask along this branch, in one message:

1. **Into an existing note?** → which one. Then only the embed line goes in, at the spot the
   user names.
2. **Or a new note — a new appendix?** → then two more things, both in the same message:
   - **Shall I create it, or will you?**
   - If you create it, you need four answers: **target folder · position in the order ·
     heading · file name**. Ask for them together, not one after the other.

This is the mistake `images.md` describes for the attachment folder, one step later. A
session saved the PDF correctly, then decided on its own that the note must not be created
and explained the Obsidian route instead; the user had to ask a second time before the file
appeared (measured 15.08.2026). Saving the attachment and then stopping is the worst of both
— the work is half done and the user carries the rest.

## Syntax

```markdown
![[appendix.pdf]]
![[appendix.pdf|Declaration of Lima]]
```

Same form as an image embed: the file must be in the vault, and the embed must stand **on a
line of its own** (text directly above or below is fine since 24.10.2026).

**A caption after the pipe is parsed but usually thrown away.** It fills the `%caption%`
placeholder of `pdfCmdText` — and the standard template is `\includepdf[pages=-]{%path%}`,
which has no `%caption%` in it. So the text silently disappears. Either leave it out, or add
`%caption%` to the template (see below).

## What it produces

`\includepdf[pages=-]{…}` from the `pdfpages` package, shaped by the DDS field `pdfCmdText`.
`pages=-` takes **all** pages. On export the file lands in a `pdf/` subfolder.

`pdfpages` is in the standard preamble — **no new package**. Full field catalogue: `dds.md`.

**Changing the command for one embed** — only some pages, landscape, anything else in the
brackets — works the same way in every case: a ` ```dds ` block before the embed and a second
one after it that puts the value back. `pdfCmdText` applies to every PDF in the document, so
the second block is what keeps the change local. The pattern is written out under "Embedding
landscape" below. **Some pages only** is the same sandwich with a range or a list in braces:
`pages={3-7}`, `pages={1,4,9}`, also mixed `pages={1-3,7}`; the braces are needed as soon as
there is a comma. It combines with the layout options below (`pages={1-3},nup=1x2,…`).
Measured 24.10.2026 through the converter: a range, a list, and a range with two pages per
page each brought exactly the pages named, 0 errors.

**Prefer that over a raw ` ```latex ` block with the `\includepdf` line written out.** The raw
block reaches the same PDF, but it costs the preview: Obsidian then shows LaTeX source where
the author expects the embedded document. With the `dds` sandwich the `![[…pdf]]` line stays
in the note and keeps being rendered.

## Portrait or landscape — measure before embedding

**Check the page orientation of every PDF before you embed it.** `\includepdf[pages=-]`
scales each source page onto the document's portrait page. A landscape source therefore lands
as a narrow strip in the middle of a portrait page, far too small to read — and nothing warns
about it. The author sees it only in the finished PDF.

### The measurement

The page geometry sits in the PDF's `/MediaBox` entries (`x0 y0 width height`) and its
`/Rotate` entries. Both can be read out of the raw file, with nothing installed:

```
powershell.exe -NoProfile -Command "$t=[IO.File]::ReadAllText('<path>',[Text.Encoding]::GetEncoding(28591)); $o=[regex]::Matches($t,'/MediaBox\s*\[\s*[\d.-]+\s+[\d.-]+\s+([\d.-]+)\s+([\d.-]+)')|%{$w=[double]$_.Groups[1].Value;$h=[double]$_.Groups[2].Value;if($w -gt $h){'landscape'}elseif($h -gt $w){'portrait'}else{'square'}}; $r=[regex]::Matches($t,'/Rotate\s+(\d+)')|%{$_.Groups[1].Value}; ($o|group|%{'{0} x {1}' -f $_.Count,$_.Name}) -join ', '; 'Rotate: ' + (($r|group|%{'{0} x {1}' -f $_.Count,$_.Name}) -join ', ')"
```

It answers in two lines, e.g. `3 x landscape` / `Rotate: ` — the tally of page shapes, then
the tally of rotations. **In the Bash tool the `$` must be escaped** (`\$t`), same as for the
clipboard command in `images.md`.

Verified 15.08.2026 against five files — LaTeX, Word, PowerPoint and two scans: all-landscape,
all-portrait, mixed and rotated were each reported correctly.

### Reading the answer

- **`/Rotate 90` or `270` swaps width and height.** It is the number of degrees the viewer
  turns the page before showing it, so a page can measure landscape and *be* portrait. A
  194-page scan measured `841.92 × 595.32` — landscape by the numbers — with `/Rotate 270` on
  every page; rendered, it is portrait, and `\includepdf` follows the rotation and sets it
  upright on a portrait page. Deciding on the MediaBox alone would have laid that document on
  its side. `Rotate:` empty or all `0` → the numbers stand as measured.
- **The number of hits need not be the page count.** A MediaBox may be inherited by all pages,
  so a 50-page file can report one entry. Only whether the values *agree* matters.
- **Nothing measurable?** Rare — all five test files answered — but possible when the page
  tree is compressed. Do not guess then; it falls into the "ask" row below.
- The comparison assumes the thesis itself is portrait, which it almost always is. If
  `00 Document Setup.md` sets up a landscape document, the whole thing turns around.

### What to do with it

| The request, and the measurement (after the rotation correction) | What you do |
|---|---|
| the request names a layout (the page, how many PDF pages on each, or both) | write it, see "Layouts, by request"; no question |
| no layout named, every page portrait | the standard: nothing to do, say nothing |
| no layout named, every page landscape, or mixed, square, not measurable | **ask** which layout, in one message: the page in the thesis (upright or turned) and how many PDF pages on each (one, two or four). Say in one sentence that the standard sets each wide page small on an upright page, and what reads well (measured: a landscape PDF reads well turned with one per page, or upright with two per page) |

**Never turn a PDF unasked** (since 24.10.2026, wish of the developer: an author may want the
thesis upright, with two wide pages on each). Until then this table said to embed landscape
without asking.

### Layouts, by request (since 24.10.2026)

The Blueprints cards "Add a PDF as an appendix" (`embed-pdf`) and "Change how an embedded PDF is
laid out" (`pdf-layout`) carry two choices. Each clicked choice adds one sentence; match it to
its row and write those options into the brackets of `\includepdf`, nothing else:

| The request says | In the brackets of `\includepdf` |
|---|---|
| Use upright pages for it. | no `landscape` (the standard) |
| Use pages turned to landscape for it. | `landscape=true` |
| Put one PDF page on each page. | no `nup` (the standard) |
| Put two PDF pages on each page. | `nup=1x2` on upright pages (one above the other), `nup=2x1` on turned pages (side by side), each with `delta=5mm 5mm` |
| Put four PDF pages on each page. | `nup=2x2,delta=5mm 5mm` |

- **Both sentences combine:** `\\includepdf[pages=-,nup=2x1,delta=5mm 5mm,landscape=true]{%path%}`
  for turned pages with two on each (written as it stands in a `dds` block).
- **Only one sentence:** the other side stays as the embed has it. A page range from "Embed only
  some pages" stays too.
- **Reach**, as everywhere: one embed, a ` ```dds ` pair around it; every embed in the part I
  marked, the first block where the part begins and the second where it ends (with no second
  marker, the first block alone); every embed in the manuscript, `pdfCmdText` in
  `00 Document Setup.md` (pairs for single embeds go, say how many).
- **Say it when pages get small,** in one sentence: measured, a landscape PDF on upright pages
  with one per page, a portrait PDF on turned pages with one per page, and four per page with
  any PDF.

Measured 24.10.2026 with a four-page landscape and a four-page portrait PDF: all six
combinations, 0 errors; the layouts also through the converter with `dds` blocks.

### Embedding landscape

````markdown
```dds
{"pdfCmdText":"\\includepdf[pages=-,landscape=true]{%path%}"}
```

![[survey.pdf]]

```dds
{"pdfCmdText":"\\includepdf[pages=-]{%path%}"}
```
````

- **The second block is not optional.** A `dds` block is a patch that holds from its position
  to the end of the document. Without the reset every later PDF in the thesis comes out
  landscape too — silently, because nothing about the second file looks wrong.
- **Both blocks produce nothing in the PDF.** In Obsidian they show as two small code blocks
  — but the `![[…pdf]]` line between them keeps rendering as the embedded document. That is
  the whole advantage over a raw ` ```latex ` block, which replaces the preview of the
  document itself with source code (see "What it produces").
- **Note the doubled backslash.** The block is JSON, so `\includepdf` is written `\\includepdf`.
  A single backslash is invalid JSON, and an invalid block is dropped without a word — the
  old value simply stays in force.
- **What `landscape=true` does**, measured: the sheet stays A4, the page gets `/Rotate 90`.
  The reader shows that page turned, and on paper it is an A4 sheet read sideways. The rest of
  the thesis is untouched.
- **The chapter's own heading page stays upright.** `# Appendix A` is an ordinary portrait
  page; the tilt starts with the first embedded page. Say this in advance — it looks like a
  half-applied setting otherwise.

## Traps

- **An embedded PDF brings its own page layout.** It is inserted as full pages, so margins,
  running headers and page numbers of the thesis do **not** apply to it. For a wide appendix
  that is usually what you want; if the user expects their header to continue, say that it
  will not.
- **A PDF cannot be referenced.** An embed gets no `\label`, so `\vref` cannot reach it and it
  appears in no list. Link to the heading above it instead. (Tables and figures with a caption
  do get a label and can be linked to, see `tables.md` and `images.md`.)
- **Two PDFs with the same file name in different folders** resolve differently in Obsidian and
  Obsitex — write the path from the vault root (`![[appendix/survey.pdf]]`). On export both are
  kept apart automatically so neither overwrites the other, but the *wrong one* may still be
  embedded.
- **Never drop the extension.** Obsitex strips extensions when matching and may embed a PDF
  where a picture was meant, or the reverse.
