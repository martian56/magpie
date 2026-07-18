# magpie

[![CI](https://github.com/martian56/magpie/actions/workflows/ci.yml/badge.svg)](https://github.com/martian56/magpie/actions/workflows/ci.yml)

Markdown rendered into styled terminal lines for
[Raven](https://github.com/martian56/raven). Give it a markdown string, a
width, and a theme; it hands back
[plumage](https://github.com/martian56/plumage) `Line`s, wrapped, styled,
and ready to draw.

Built for showing model and tool output in terminal UIs (it is what the
[rook](https://github.com/martian56/rook) coding agent renders replies
with), and useful anywhere a plumage app wants formatted text.

## Install

```toml
[dependencies]
"github.com/martian56/magpie" = "v0.1.0"
"github.com/martian56/plumage" = "v0.2.0"
```

plumage provides the `Line`, `Span`, and `Style` types the rendering is
made of; your code usually imports those too.

## Usage

```rust
import "github.com/martian56/magpie" { render, plain }
import "github.com/martian56/magpie/theme" { MdTheme, default_theme }
import "github.com/martian56/plumage/widgets/paragraph" { Paragraph }

let lines = render(markdown, 72, default_theme())
Paragraph.from_lines(lines).render(area, f.buf)
```

`render(md, width, theme) -> List<Line>` is the whole API, plus
`plain(line) -> String` for the unstyled text of a rendered line (selection
copy, tests).

## The dialect

The line-oriented subset that model and tool output actually uses:

- Headings (`#` through `######`), styled and stripped of their marks.
- Paragraphs, word-wrapped to the width, with `**bold**`, `*italic*`,
  `` `inline code` ``, and `[text](url)` links (label styled, URL shown
  after it).
- Fenced code blocks (```` ``` ```` or `~~~`), verbatim under a two-column
  indent, long lines hard-split so nothing overflows.
- Bullet lists (`-`, `*`, `+`) and numbered lists, wrapped with hanging
  indents; nesting by indentation.
- Blockquotes with a bar, and horizontal rules spanning the width.
- Blank lines collapse; words wider than a line (paths, URLs) hard-split.

Deliberate choices for terminal output of code-heavy text:

- Underscores are always literal, so `snake_case` never italicizes.
- A single `*` toggles italic only when it hugs a word; `2 * 3` stays as
  written.
- An unclosed backtick or fence renders as code to the end of the input,
  which is exactly what a streaming reply looks like before its closing
  marker arrives, so mid-stream rendering stays sensible.

Out of scope for now: tables, setext headings, indented (four-space) code
blocks, images, HTML, and escape sequences.

## Theming

`MdTheme` is one style per element: `text`, `heading`, `bold`, `italic`,
`code`, `code_block`, `bullet`, `quote`, `rule`, `link`, `url`.
`default_theme()` uses only attributes (bold, dim, underline), no colors,
so it reads on any background; build your own from your palette:

```rust
import "github.com/martian56/plumage/style" { Style, Color }

let th = default_theme()
th.heading = Style.new().fg(Color.Cyan).bold()
th.code = Style.new().fg(Color.Yellow)
```

## Demo

```
rvpm run
```

renders a sample document at 60 columns and prints the plain text, so the
layout can be eyeballed without a terminal UI. Styling is covered by the
test suite, which asserts on the spans of rendered lines; everything runs
without a terminal.

## Layout

- `magpie` (lib.rv): `render` and `plain`, the block-level pass (headings,
  fences, lists, quotes, rules, paragraphs).
- `magpie/theme`: `MdTheme` and `default_theme`.
- `magpie/inline`: the inline parser (`parse_inline`) and the span-aware
  word wrap (`wrap_spans`), importable on their own for custom layouts.

## Development

```
rvpm build    # type-check the library, build the demo
rvpm test     # rendering tests, no terminal needed
rvpm fmt
```

## License

MIT
