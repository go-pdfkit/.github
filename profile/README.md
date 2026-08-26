<p align="center"><img src="https://raw.githubusercontent.com/go-pdfkit/.github/main/assets/banner.svg" alt="go-pdfkit" width="800"></p>

<h1 align="center">go-pdfkit</h1>
<p align="center"><strong>The whole of PDF in pure Go, with no C anywhere.</strong></p>
<p align="center">Read a file, write one, rearrange it, draw it, read it back as words and pictures,<br>and edit it with other people — in a browser tab if you like.</p>

<p align="center">
  🌐 <a href="https://go-pdfkit.github.io">Website</a> ·
  📚 <a href="https://go-pdfkit.github.io/docs/">Documentation</a>
</p>

<p align="center">
  <a href="https://go-pdfkit.github.io/docs/"><img alt="Docs" src="https://img.shields.io/badge/docs-mkdocs--material-991B1B?style=flat-square"></a>
  <a href="https://github.com/go-pdfkit/pdfkit/blob/main/LICENSE"><img alt="License: BSD-3-Clause" src="https://img.shields.io/badge/license-BSD--3--Clause-blue?style=flat-square"></a>
  <img alt="Go 1.26.4+" src="https://img.shields.io/badge/go-1.26.4%2B-00ADD8?style=flat-square&logo=go&logoColor=white">
  <img alt="Coverage 100%" src="https://img.shields.io/badge/coverage-100%25-1a7f37?style=flat-square">
</p>

---

go-pdfkit is the whole of PDF in Go with **zero C dependencies** anywhere
(`CGO_ENABLED=0`), and every library here builds for `GOOS=js/wasm`. Fonts are
parsed and shaped by [go-opentype](https://github.com/go-opentype/opentype);
pages are rasterised by [go-gfx](https://github.com/go-gfx/gfx).

```go
// Take three pages out of a report, turn one the right way up, stamp the lot.
doc, _ := ops.Open(bytes)
doc.Select("4-6")
doc.SetRotation("2", 90)
doc.Watermark("all", "DRAFT")
out, _ := doc.Bytes()

// Draw a page, and read one back.
img, _ := render.Page(src, 1, render.Options{DPI: 150})
text, _ := extract.Text(src, 1)
```

```
$ pdfops merge out.pdf a.pdf b.pdf
$ pdfops nup -n 4 slides.pdf handout.pdf
$ pdfops encrypt -user letmein -allow print,copy plain.pdf locked.pdf
$ pdfops text -layout -pages 1 paper.pdf
```

## Repositories

| Repo | What it is |
|------|------------|
| [**reader**](https://github.com/go-pdfkit/reader) | reads and writes the format itself: objects, xref tables *and* streams, object streams, repair, every filter, encryption to AES-256, content streams, the page tree — and a writer that produces the same bytes twice |
| [**ops**](https://github.com/go-pdfkit/ops) | the verbs, and the `pdfops` command: merge, split, rotate, crop, n-up, booklet, watermark, stamp, sanitize, compress, encrypt, and reading a page back |
| [**render**](https://github.com/go-pdfkit/render) | turns a page into pixels: paths, images, text in every font flavour, PDF functions, all seven shadings and tiling patterns |
| [**pdffont**](https://github.com/go-pdfkit/pdffont) | what a document says about a font — encodings, widths, and what text a code stands for |
| [**extract**](https://github.com/go-pdfkit/extract) | reads a page back: the text with where it sits, and the pictures with the box each covers |
| [**coedit**](https://github.com/go-pdfkit/coedit) | a PDF several people edit at once — the plan is shared, not the file |
| [**app**](https://github.com/go-pdfkit/app) | a PDF workbench that runs in a browser tab and nowhere else |
| [**pdfkit**](https://github.com/go-pdfkit/pdfkit) | the document builder: pages, vector graphics, and text in embedded subsetted fonts |
| [**docs**](https://github.com/go-pdfkit/docs) | the documentation site (MkDocs Material, versioned with mike) |
| [**go-pdfkit.github.io**](https://github.com/go-pdfkit/go-pdfkit.github.io) | the org landing page (Hugo) |

## Measured against 118 863 real files

Not against files written to pass a test. The corpus is arXiv's figures — from
Matplotlib, Mathematica, pdfTeX, Ghostscript and Adobe — and every wave of work
is checked by **hashing the pixels of every page before and after** and putting
the biggest changes beside what the operating system's own renderer draws.

| | |
|---|---|
| files that open | **118 833** of 118 835 (the rest are PNGs named `.pdf`) |
| content operations read | 1 536 769 753 |
| rewritten to an identical fingerprint | 118 833 of 118 833 |
| compressed, page for page identical | 118 833 — 35.2 GB → 33.6 GB |
| encrypted and decrypted both ways | 118 833 |
| pages drawn, no panics | 4 108 |
| pages read back as text | 121 946 — 34 million characters |
| pictures located | 300 685 |

That is what found a stroke that came out at **half its colour**, a font that
took the dots off every *i*, a colour transform that turned every plot's paper
**yellow**, a pattern that rubbed the page out, and every gradient inside a
figure landing **in the corner of the page** rather than on the panel it was
meant to fill. Each is written up in the release notes of the library it was
found in.

## Honest about scope

- An ExtGState **soft mask** — a luminosity mask, the way Illustrator writes a
  faded surface — is not applied, so a page that uses one comes out with that
  part drawn at full strength. Constant alpha, `/ca` and `/CA`, is honoured.
- A page whose text the document gives **no way to read** — a Type 3 dvips
  font with no `/ToUnicode`, whose glyphs are called `a1` — comes back marked
  unreadable rather than guessed at. That is 1.89% of runs.
- Eight of 14 614 embedded Type 1 programs have a private half that decrypts
  cleanly for eighty bytes and then does not: one byte of each was altered
  before it was embedded, and no reader can recover that.
- Tagged PDF and PDF/A, forms and interactive annotations are not implemented.

## Principles

- **Pure Go, zero cgo, zero C dependencies.** No bundled `libpoppler`,
  `libharfbuzz` or `libfreetype` — cross-compiles anywhere Go runs, and all of
  it runs in a browser tab.
- **Built on go-opentype**, not a second font stack: sfnt parsing and
  complex-script shaping (GSUB/GPOS) come from
  [go-opentype/opentype](https://github.com/go-opentype/opentype), the same
  engine behind [go-opentype](https://github.com/go-opentype) and
  [go-ruby-prawn](https://github.com/go-ruby-prawn).
- **Correctness checked against an independent parser** — generated
  documents are re-opened with [`rsc.io/pdf`](https://pkg.go.dev/rsc.io/pdf)
  and their structure verified; the embedded TrueType subset is re-parsed
  with go-opentype to confirm it still contains the glyphs that were drawn.
- **100% test coverage**, deterministic and network-free — tests use a
  synthesised TrueType font and a bundled OFL OpenType/CFF font, never a
  network fetch.

## Status

Every library here is released, runs at **100% statement coverage** including
the branches that report a file saying something it should not, and builds for
the six 64-bit architectures the fleet targets plus `js/wasm`, macOS and
Windows.

BSD-3-Clause.
