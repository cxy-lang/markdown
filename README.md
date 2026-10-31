# markdown

**A fast, feature-rich Markdown to HTML parser for Cxy with built-in file caching support.**

## Features

- **Fast parsing with zero-copy lexing** — efficient tokenization without unnecessary string allocations
- **File caching to avoid re-rendering unchanged documents** — significant performance boost for static site generation and documentation builds
- **Semantic HTML with consistent `mk-*` class names** — easy to style with CSS while maintaining clean, semantic markup
- **CommonMark support** — headings, lists, code blocks, tables, blockquotes, links, images, and more
- **Flexible output** — render to strings, files, or any `OutputStream` for maximum flexibility
- **Non-blocking I/O option** — suitable for async/coroutine contexts with configurable I/O modes

## Installation

Add the markdown package to your Cxy project:

```markdown/README.md#L1-2
import { Markdown, MarkdownConfig } from "@markdown"
```

## Quick Start

### Basic Usage

The simplest way to convert Markdown to HTML is using the `toHtml()` method with a string:

```markdown/README.md#L1-5
import { Markdown } from "@markdown"

var md = Markdown()
md.toHtml(&stdout, "# Hello **World**".s)
```

This renders directly to stdout, but you can use any `OutputStream`:

```markdown/README.md#L1-6
var md = Markdown()
var output = String()
md.toHtml(&output, "# Hello **World**\n\nThis is a paragraph.".s)
println(output)
```

### File Input

Read and render a Markdown file directly:

```markdown/README.md#L1-7
import { Markdown } from "@markdown"
import { Path } from "stdlib/path.cxy"

var md = Markdown()
var output = String()
md.toHtml(&output, Path("docs/guide.md"))
```

### File Caching

For maximum performance when processing multiple files or rebuilding documentation, use the file-to-file mode with caching:

```markdown/README.md#L1-11
import { Markdown, MarkdownConfig } from "@markdown"
import { Path } from "stdlib/path.cxy"

var config = MarkdownConfig{ cacheDir: Path(".markdown-cache") }
var md = Markdown(&&config)

// First call parses and caches
var htmlPath = md.toHtml(Path("docs/guide.md"))
// Returns: Path(".markdown-cache/docs/guide.html")

// Subsequent calls return cached file if source hasn't changed
var cachedPath = md.toHtml(Path("docs/guide.md"))  // Instant!
```

## API Reference

### `Markdown` Class

The main parser and renderer class.

#### Constructors

```markdown/README.md#L1-2
func `init`()
```

Creates a Markdown instance with default configuration (cache directory `.markdown`).

```markdown/README.md#L1-2
func `init`(config: MarkdownConfig)
```

Creates a Markdown instance with custom configuration.

#### Methods

##### `toHtml(os: &OutputStream, source: strview): !void`

Parse a Markdown string and write HTML to an output stream. No caching — the string is always re-parsed.

**Parameters:**
- `os` — Output stream to write HTML to (e.g., `&stdout`, `&String`, `&FileOutputStream`)
- `source` — Markdown source string

**Example:**
```markdown/README.md#L1-4
var md = Markdown()
var output = String()
md.toHtml(&output, "# Heading\n\nParagraph text.".s)
```

##### `toHtml(os: &OutputStream, path: PathLike): !void`

Read a Markdown file, parse it, and write HTML to an output stream. No caching — the file is always re-read and re-parsed.

**Parameters:**
- `os` — Output stream to write HTML to
- `path` — Path to the Markdown file

**Example:**
```markdown/README.md#L1-5
var md = Markdown()
var output = String()
md.toHtml(&output, Path("README.md"))
println(output)
```

##### `toHtml(path: PathLike): !Path`

Convert a Markdown file to an HTML file under the configured cache directory. Returns the path of the generated (or cached) HTML file.

**Caching behavior:** If `config.noCache` is `false` and the cached output exists and is newer than the source file, returns the cached path immediately without re-rendering.

**Parameters:**
- `path` — Path to the Markdown source file

**Returns:** Path to the generated HTML file in the cache directory

**Example:**
```markdown/README.md#L1-5
var md = Markdown()
var htmlPath = md.toHtml(Path("docs/guide.md"))
// => Path(".markdown/docs/guide.html")
println(f"HTML saved to: {htmlPath}")
```

### `MarkdownConfig` Struct

Configuration options for `Markdown` instances.

#### Fields

##### `cacheDir: Path`

Directory used for file-to-HTML cache output. The cache mirrors the structure of your source files.

**Default:** `Path(".markdown")`

**Example:**
```markdown/README.md#L1-2
var config = MarkdownConfig{ cacheDir: Path("build/html-cache") }
var md = Markdown(&&config)
```

##### `noCache: bool`

If `true`, the file-to-HTML method always re-renders even when the cached output is newer than the source.

**Default:** `false`

**Example:**
```markdown/README.md#L1-3
// Force fresh renders every time
var config = MarkdownConfig{ noCache: true }
var md = Markdown(&&config)
```

##### `nonBlocking: bool`

If `true`, file I/O is opened in non-blocking mode, suitable for use inside coroutine or async contexts.

**Default:** `false`

**Example:**
```markdown/README.md#L1-2
var config = MarkdownConfig{ nonBlocking: true }
var md = Markdown(&&config)
```

## Supported Markdown Features

The parser supports a comprehensive set of CommonMark features:

### Text Formatting

- **Headings** (`# H1` through `###### H6`) — generates `<h1 class="mk-h1">` through `<h6 class="mk-h6">`
- **Bold** (`**text**` or `__text__`) — generates `<strong class="mk-strong">`
- **Italic** (`*text*` or `_text_`) — generates `<em class="mk-em">`
- **Inline code** (`` `code` ``) — generates `<code class="mk-code">`
- **Strikethrough** (`~~text~~`) — generates `<del class="mk-del">`

### Code Blocks

Fenced code blocks with optional language specifiers:

````markdown/README.md#L1-5
```rust
fn main() {
    println!("Hello");
}
```
````

Generates `<pre class="mk-pre mk-pre--rust">` with `<code class="mk-code language-rust">`.

### Lists

- **Unordered lists** (`-`, `*`, or `+`) — generates `<ul class="mk-ul">` with `<li class="mk-li">`
- **Ordered lists** (`1.`, `2.`, etc.) — generates `<ol class="mk-ol">` with `<li class="mk-li">`
- Supports nested lists

### Blockquotes

```markdown/README.md#L1-2
> Quoted text
> Multiple lines
```

Generates `<blockquote class="mk-blockquote">`.

### Links and Images

- **Links** (`[text](url)`) — generates `<a class="mk-a" href="url">`
- **Images** (`![alt](src)`) — generates `<img class="mk-img" src="src" alt="alt">`

### Tables

```markdown/README.md#L1-4
| Name  | Age |
|-------|-----|
| Alice | 30  |
| Bob   | 25  |
```

Generates semantic table markup:
- `<table class="mk-table">`
- `<thead class="mk-thead">` with `<th class="mk-th">`
- `<tbody class="mk-tbody">` with `<td class="mk-td">`

### Other Elements

- **Horizontal rules** (`---`, `***`, or `___`) — generates `<hr class="mk-hr">`
- **Paragraphs** — generates `<p class="mk-p">`

### CSS Class Reference

All generated HTML elements include consistent `mk-*` class names for easy styling:

| Element | Class Names |
|---------|-------------|
| Headings | `mk-h1`, `mk-h2`, `mk-h3`, `mk-h4`, `mk-h5`, `mk-h6` |
| Paragraph | `mk-p` |
| Bold | `mk-strong` |
| Italic | `mk-em` |
| Inline code | `mk-code` |
| Code block | `mk-pre`, `mk-pre--{lang}`, `mk-code`, `language-{lang}` |
| Lists | `mk-ul`, `mk-ol`, `mk-li` |
| Blockquote | `mk-blockquote` |
| Link | `mk-a` |
| Image | `mk-img` |
| Table | `mk-table`, `mk-thead`, `mk-tbody`, `mk-th`, `mk-td` |
| Horizontal rule | `mk-hr` |
| Document wrapper | `mk-doc` |

## HTML Output

### Document Wrapper

All rendered HTML is wrapped in a `<div class="mk-doc">` container:

```markdown/README.md#L1-5
<div class="mk-doc">
  <h1 class="mk-h1">Your Content</h1>
  <p class="mk-p">Paragraph text...</p>
</div>
```

This makes it easy to scope your styles and distinguish Markdown-generated content from other HTML on the page.

### HTML Escaping

All text content is properly HTML-escaped to prevent XSS vulnerabilities:

- `<` becomes `&lt;`
- `>` becomes `&gt;`
- `&` becomes `&amp;`
- `"` becomes `&quot;`
- `'` becomes `&#39;`

Code blocks preserve all characters safely while maintaining proper syntax highlighting compatibility.

## Styling

The consistent `mk-*` class names make styling straightforward. Here's a starter CSS theme:

```markdown/README.md#L1-68
/* Base document styles */
.mk-doc {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  line-height: 1.6;
  color: #333;
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

/* Headings */
.mk-h1, .mk-h2, .mk-h3, .mk-h4, .mk-h5, .mk-h6 {
  margin-top: 1.5em;
  margin-bottom: 0.5em;
  font-weight: 600;
  line-height: 1.25;
}

.mk-h1 { font-size: 2.5em; border-bottom: 2px solid #eee; padding-bottom: 0.3em; }
.mk-h2 { font-size: 2em; border-bottom: 1px solid #eee; padding-bottom: 0.3em; }
.mk-h3 { font-size: 1.5em; }
.mk-h4 { font-size: 1.25em; }
.mk-h5 { font-size: 1em; }
.mk-h6 { font-size: 0.875em; color: #666; }

/* Paragraphs */
.mk-p {
  margin: 1em 0;
}

/* Text formatting */
.mk-strong { font-weight: 600; }
.mk-em { font-style: italic; }

/* Code */
.mk-code {
  background: #f6f8fa;
  padding: 0.2em 0.4em;
  border-radius: 3px;
  font-family: "SF Mono", Monaco, Consolas, monospace;
  font-size: 0.9em;
}

.mk-pre {
  background: #f6f8fa;
  border-radius: 6px;
  padding: 1em;
  overflow-x: auto;
  margin: 1em 0;
}

.mk-pre .mk-code {
  background: transparent;
  padding: 0;
}

/* Lists */
.mk-ul, .mk-ol {
  margin: 1em 0;
  padding-left: 2em;
}

.mk-li {
  margin: 0.25em 0;
}

/* Blockquotes */
.mk-blockquote {
  border-left: 4px solid #ddd;
  padding-left: 1em;
  margin: 1em 0;
  color: #666;
}

/* Links */
.mk-a {
  color: #0366d6;
  text-decoration: none;
}

.mk-a:hover {
  text-decoration: underline;
}

/* Images */
.mk-img {
  max-width: 100%;
  height: auto;
  display: block;
  margin: 1em 0;
}

/* Tables */
.mk-table {
  border-collapse: collapse;
  width: 100%;
  margin: 1em 0;
}

.mk-th, .mk-td {
  border: 1px solid #ddd;
  padding: 0.5em 1em;
  text-align: left;
}

.mk-th {
  background: #f6f8fa;
  font-weight: 600;
}

/* Horizontal rule */
.mk-hr {
  border: none;
  border-top: 2px solid #eee;
  margin: 2em 0;
}
```

You can customize these styles or add syntax highlighting by targeting language-specific classes like `.mk-pre--rust` or `.language-rust`.

## Caching System

The caching system is designed for efficient rebuilds of documentation sites and multi-file Markdown projects.

### How It Works

When you call `toHtml(path)` with a file path, the markdown package:

1. **Checks the cache:** Looks for a cached HTML file in `cacheDir`
2. **Compares timestamps:** If the cached file exists and is newer than the source, returns the cached path immediately
3. **Renders if needed:** If no cache exists or source is newer, re-renders the Markdown and updates the cache
4. **Returns path:** Returns the path to the HTML file (cached or newly generated)

### Directory Structure

The cache directory mirrors your source file structure. For example:

```markdown/README.md#L1-9
Source files:              Cache output:
docs/guide.md       →      .markdown/docs/guide.html
docs/api/intro.md   →      .markdown/docs/api/intro.html
README.md           →      .markdown/README.html
```

This makes it easy to:
- Find cached files that correspond to source files
- Deploy the entire cache directory as static HTML
- Clean up by simply deleting the cache directory

### Example Directory Layout

```markdown/README.md#L1-15
my-project/
├── docs/
│   ├── getting-started.md
│   ├── api-reference.md
│   └── examples.md
├── .markdown/              # Default cache directory
│   └── docs/
│       ├── getting-started.html
│       ├── api-reference.html
│       └── examples.html
└── main.cxy
```

### Disabling the Cache

To force fresh renders on every call (useful during development):

```markdown/README.md#L1-4
var config = MarkdownConfig{ noCache: true }
var md = Markdown(&&config)
// Now every call re-renders from scratch
```

### Clearing the Cache

Simply delete the cache directory:

```markdown/README.md#L1-2
# On Unix-like systems
rm -rf .markdown/
```

Or programmatically in Cxy using stdlib path utilities.

## Error Handling

The `toHtml()` methods return `!void` or `!Path`, indicating they may throw errors.

Common error scenarios:

- **File not found:** Thrown when trying to render a non-existent file
- **I/O errors:** Thrown when file read/write operations fail
- **Permission errors:** Thrown when lacking permissions to read source or write cache

Handle errors using Cxy's error handling mechanisms:

```markdown/README.md#L1-9
import { Markdown } from "@markdown"
import { Path } from "stdlib/path.cxy"

var md = Markdown()
try {
    var htmlPath = md.toHtml(Path("docs/guide.md"))
    println(f"Success: {htmlPath}")
} catch (err) {
    eprintln(f"Error rendering markdown: {err}")
}
```

## Testing

The markdown package includes comprehensive tests covering all supported features.

### Run the Test Suite

```markdown/README.md#L1-2
cxy package test
```

This runs all unit tests defined in the source files.

### Run the Example

To see the markdown parser in action with various features:

```markdown/README.md#L1-2
cxy run examples/markdown.cxy
```

This example demonstrates:
- String-to-HTML conversion
- File-to-HTML conversion
- Caching behavior
- Various Markdown features

The example creates a `sample.md` file and a `markdown-cache/` directory that you can inspect.

## License

This project is licensed under the terms specified in the [LICENSE](LICENSE) file.

---

**Made with ❤️ for the Cxy programming language**
