<p align="center"><img src="https://raw.githubusercontent.com/go-pdfkit/.github/main/assets/banner.svg" alt="go-pdfkit" width="800"></p>

<h1 align="center">go-pdfkit</h1>
<p align="center"><strong>Pure-Go, CGO-free PDF 1.7 writer with a Go-idiomatic API.</strong></p>

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

go-pdfkit builds **PDF 1.7** documents from Go: pages, vector graphics, and
text set in embedded, subsetted fonts — with **zero C dependencies**
anywhere in the stack (`CGO_ENABLED=0`). Fonts are parsed and shaped by
[go-opentype](https://github.com/go-opentype/opentype), the same pure-Go
text stack behind the [go-opentype](https://github.com/go-opentype) org and
the Ruby port [go-ruby-prawn](https://github.com/go-ruby-prawn).

```go
ttf, _ := os.ReadFile("font.ttf")
font, _ := pdfkit.LoadFont(ttf)

doc := pdfkit.New(pdfkit.Options{Title: "Hello"})
p := doc.AddPage(pdfkit.A4)

p.SetFont(font, 24)
p.Text(pdfkit.Mm(20), p.Height()-pdfkit.Mm(20), "Hello, pdfkit")

f, _ := os.Create("out.pdf")
defer f.Close()
doc.Write(f)
```

## Repositories

| Repo | What it is |
|------|------------|
| [**pdfkit**](https://github.com/go-pdfkit/pdfkit) | the writer — document/page tree/xref (`document.go`), vector graphics (`page.go`), text (`text.go`), font embedding (`embed.go`, `subset.go`), images (`image.go`) |
| [**docs**](https://github.com/go-pdfkit/docs) | this documentation site (MkDocs Material, versioned with mike) |

## What it does

- **Documents** — catalog, page tree, cross-reference table and trailer;
  writes to any `io.Writer`; optional FlateDecode stream compression.
- **Graphics** — paths, fill & stroke (nonzero and even-odd), DeviceGray/RGB/
  CMYK colour, line width/cap/join/miter/dash, CTM transforms, clipping,
  `q`/`Q` state, constant-alpha transparency via ExtGState.
- **Text** — fonts embedded as **Type0 (Identity-H)** composite fonts with
  glyph **subsetting**, a per-glyph `/W` width array and a `/ToUnicode` CMap
  for copy/paste. TrueType `glyf` → **FontFile2 / CIDFontType2**; CFF/
  OpenType → **FontFile3 / CIDFontType0**. Char/word spacing, leading,
  render modes, greedy text wrapping, and an optional **shaped-text** API
  (GSUB/GPOS via go-opentype) for Arabic/Indic/CJK.
- **Images** — JPEG embedded directly (DCTDecode, no re-encoding); PNG and
  any `image.Image` rasterised as XObjects (FlateDecode) with an `/SMask`
  for alpha.
- **Deterministic** — with the zero `Options`, output has no timestamps and
  a content-derived `/ID`, so identical inputs produce byte-identical PDFs.

## Honest about scope

- CFF/OpenType fonts embed their **whole `CFF ` table** — charstring
  subsetting is not yet implemented. TrueType fonts **are** fully
  glyph-subsetted.
- go-opentype exposes no raw table bytes, units-per-em, glyf/loca arrays or
  a subsetting export, so pdfkit reparses the sfnt container it is handed
  and implements TrueType subsetting itself.
- Encryption, tagged PDF / PDF-A, forms and annotations are out of scope for
  v0.1.

## Principles

- **Pure Go, zero cgo, zero C dependencies.** No bundled `libpoppler`,
  `libharfbuzz` or `libfreetype` — cross-compiles anywhere Go runs.
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

`pdfkit` v0.1.0: PDF 1.7 documents, vector graphics, embedded/subsetted
TrueType and CFF fonts, shaped text, JPEG/PNG images. 100% statement
coverage, `CGO_ENABLED=0`.

BSD-3-Clause.
