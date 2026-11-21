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
"github.com/martian56/magpie" = "v0.4.0"
"github.com/martian56/plumage" = "v0.4.0"
```

plumage provides the `Line`, `Span`, and `Style` types the rendering is
made of; your code usually imports those too.

## Usage

```rust
from "github.com/martian56/magpie" import render, plain
from "github.com/martian56/magpie/theme" import MdTheme, default_theme
from "github.com/martian56/plumage/widgets/paragraph" import Paragraph

let lines = render(markdown, 72, default_theme())
Paragraph.from_lines(lines).render(area, f.buf)
```

`render(md, width, theme) -> list[Line]` is the whole API, plus
`plain(line) -> str` for the unstyled text of a rendered line (selection
copy, tests).

## The dialect

The line-oriented subset that model and tool output actually uses:

- Headings (`#` through `######`), styled and stripped of their marks.
- Paragraphs, word-wrapped to the width, with `**bold**`, `*italic*`,
  `~~strikethrough~~`, `` `inline code` ``, and `[text](url)` links (label
  styled, URL shown after it).
- Fenced code blocks (```` ``` ```` or `~~~`), verbatim under a two-column
  indent, long lines hard-split so nothing overflows; a ```` ```diff ````
  fence styles its added and removed lines.
- Bullet lists (`-`, `*`, `+`) and numbered lists, wrapped with hanging
  indents; nesting by indentation. Task items (`- [x]`, `- [ ]`) render as
  checkboxes.
- Pipe tables: columns sized to their widest cell, the `---`/`:---:`
  separator row consumed for left, right, or center alignment and drawn as
  a rule under the bold header; cells parse inline markup; a table wider
  than the width clips instead of breaking its grid; a backslash-escaped
  pipe is cell content, not a boundary.
- Blockquotes with a bar that nest full block markdown (headings, lists,
  code, even another quote), and horizontal rules spanning the width.
- Blank lines collapse; words wider than a line (paths, URLs) hard-split.

Deliberate choices for terminal output of code-heavy text:

- Underscores are always literal, so `snake_case` never italicizes.
- A single `*` toggles italic only when it hugs a word; `2 * 3` stays as
  written.
- An unclosed backtick or fence renders as code to the end of the input,
  which is exactly what a streaming reply looks like before its closing
  marker arrives, so mid-stream rendering stays sensible.

Out of scope for now: setext headings, indented (four-space) code blocks,
images, raw HTML (a <details> block renders as its literal text), footnotes,
and general escape sequences (only | in table cells is honored).

## Theming

`MdTheme` is one style per element: `text`, `heading`, `bold`, `italic`,
`code`, `code_block`, `bullet`, `quote`, `rule`, `link`, `url`, `ins`, `del`.
`default_theme()` uses only attributes (bold, dim, underline), no colors,
so it reads on any background; build your own from your palette:

```rust
from "github.com/martian56/plumage/style" import Style, Color

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
