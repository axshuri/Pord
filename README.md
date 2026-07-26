<p align="center">
  <img src="public/logo.svg" alt="Pord logo" width="64" height="64">
</p>

# Pord — Persian Document Editor

> ویرایشگر سند فارسی

A block-based Persian (Farsi) document editor with Notion-style heading sidebar, A4 page preview, PDF and Word export with inlined fonts, auto-generated table of contents, English-to-Persian glossary detection, and drag-and-drop block management. All content is persisted locally via IndexedDB with automatic save and manual snapshots.

## Features

### Block Types

| Block | Description |
|---|---|
| **Title / Subtitle** | Document-level headings with distinct styling |
| **H2 / H3** | Section headings that auto-populate the TOC and heading sidebar |
| **Paragraph** | Standard body text with per-block font size and width control |
| **Bullet List** | Multi-item bullet lists |
| **Quote / Callout** | Block quotation and highlighted callout boxes |
| **Image** | Embedded images with caption, alt text, alignment, and width control |
| **Divider** | Horizontal separator lines |
| **Table** | Custom grid tables (add/remove rows and columns) |
| **Code** | Syntax-highlighted code blocks |
| **Spacer** | Adjustable vertical spacing blocks |
| **Columns** | Multi-column layout blocks |
| **Footnote** | Inline footnote markers with footnote text rendered below main content, linked by anchors |
| **TOC** | Auto-generated table of contents from document headings |
| **Glossary** | Auto-detected English words with Persian translations, rendered as a glossary section |
| **Page Break** | Manual page break for PDF export |
| **Bismillah** | Centered invocation block ("به نام یزدان") — large, full-line, traditional Persian document header |

### Core Features

- **Drag & Drop** — Reorder blocks via `@dnd-kit` with visual grip handles; move blocks up/down, duplicate, convert between types, merge adjacent blocks
- **Heading Sidebar** — Notion-style collapsible sidebar listing all `subtitle`, `h2`, and `h3` blocks with click-to-scroll navigation
- **A4 Page Preview** — Toggle between edit mode and a paginated A4 preview that shows how the document will look in print
- **PDF Export** | Generates a self-contained HTML document with inlined Vazirmatn fonts (base64) and base64-embedded images, then triggers browser print-to-PDF. The result matches the site's premium visual styling exactly, even when opened offline.
- **Word Export** — Builds a `.doc` file using the same HTML+CSS pipeline, with inline fonts and images, compatible with Microsoft Word and LibreOffice
- **Auto-Generated TOC** — The `toc` block dynamically collects all headings and renders them as a linked table of contents with page-style numbering
- **English Glossary** — The `glossary` block detects English words within Persian text, looks up translations, and renders a bilingual glossary table
- **Dark/Light Theme** — Toggle between a warm cream paper theme and a dark writing theme
- **Per-Block Styling** | Each block has optional font size (`sm/md/lg/xl`) and width (`full/wide/medium/narrow`) overrides, plus adjustable margins
- **Auto-Save** — Document state is auto-saved to IndexedDB (with localStorage fallback) every few seconds
- **Snapshots** — Manual named snapshots (up to 50) stored in IndexedDB; rename, delete, and restore any snapshot
- **Import/Export** — Export as `.pord.json` (full document with embedded images); import back for cross-machine round-tripping. Also supports `.txt` plain text import/export.

## Quickstart

```bash
bun install
bun run dev
```

Open `http://localhost:3000` — the editor loads with a seeded sample document.

> [!NOTE]
> No database setup required. All data is stored in the browser's IndexedDB. The Prisma/SQLite setup in `.env` is reserved for future server-side features but is not currently used by the editor.

## Architecture

```
src/
├── app/
│   ├── page.tsx                # Main editor: toolbar + block list + heading sidebar + preview toggle
│   ├── layout.tsx              # RTL layout with Vazirmatn font
│   ├── globals.css             # Editor styles, A4 preview, theme variables
│   └── api/
│       └ route.ts              # Reserved for future server-side features
├── components/
│   ├── doc/
│   │   └── BlockEditor.tsx     # Individual block component: content editing, type conversion,
│   │                           #   drag handle, action menu, per-block styling controls
│   └── ui/                     # shadcn/ui component library
├── lib/
│   ├── doc-types.ts            # Block model definitions, type conversions, TOC/glossary/footnote
│   │                           #   collection helpers, word/char counting, HTML/text parsers
│   ├── export-doc.ts           # PDF/Word export: builds self-contained HTML with inlined fonts
│   │                           #   and base64 images, A4 page dimensions, footnote/glossary rendering
│   ├── storage.ts              # IndexedDB storage layer: autosave, snapshots, import/export
│   ├── db.ts                   # Prisma client (reserved, unused currently)
│   └── utils.ts                # Utility helpers (cn, etc.)
├── hooks/
│   ├── use-toast.ts            # Toast notification hook
│   └── use-mobile.ts           # Mobile detection hook
└── public/
    ├── logo.svg                # App logo
    ├── robots.txt
    └── fonts/
        ├── Vazirmatn-Regular.woff2
        ├── Vazirmatn-Medium.woff2
        └── Vazirmatn-Bold.woff2
```

## How Export Works

### PDF Export

The export pipeline builds a self-contained HTML document that mirrors the on-screen editor styling:

1. **Font inlining** — The three Vazirmatn weight files (Regular, Medium, Bold) are loaded as base64 strings and embedded directly in the `<style>` block as `@font-face` definitions. This ensures the exported PDF renders identically even when the file is opened on a machine without Vazirmatn installed.

2. **Image inlining** — All image blocks with base64 `src` data are included inline. External URLs are also embedded. This makes the export fully self-contained — no external dependencies.

3. **A4 page layout** — The HTML is styled with A4 dimensions (210 mm × 297 mm), appropriate margins, and page-break rules for the `pageBreak` and `bismillah` blocks. The browser's print-to-PDF dialog produces paginated output.

4. **Footnote rendering** — Footnote markers appear as superscript links in the text body; footnote text is rendered as a numbered list at the bottom of the document with bidirectional anchor links.

5. **Glossary rendering** — The glossary block produces a two-column table: English term | Persian translation.

6. **TOC rendering** — The TOC block renders as a numbered list of headings with anchor links that work within the exported document.

### Word Export

The same HTML pipeline produces a `.doc` file by wrapping the styled HTML in Microsoft Word-compatible XML headers. The resulting file opens correctly in Word, LibreOffice, and Google Docs with full Persian formatting preserved.

## Block Type System

Each block follows a typed data model defined in `doc-types.ts`:

- **TextBlock** — Holds `text` content; used for title, subtitle, h2, h3, paragraph, quote, callout
- **BulletBlock** — Holds `items: string[]` array
- **ImageBlock** — Holds `src`, `caption`, `alt`, `width`, `align`
- **TableBlock** — Holds `rows: string[][]` 2D array with `headerRow` flag
- **CodeBlock** — Holds `text` content with optional `language` specification
- **ColumnsBlock** — Holds `children: Block[][]` for multi-column layout
- **FootnoteBlock** — Holds `marker` and `text` content
- **TocBlock** — Auto-generated; no editable content
- **GlossaryBlock** — Auto-detected; holds `entries: { english, persian }[]`
- **DividerBlock / SpacerBlock / PageBreakBlock / BismillahBlock** — Structural blocks with minimal data

Blocks can be converted between compatible types (e.g., paragraph → quote), merged when adjacent, and reordered via drag-and-drop.

## Storage Model

Documents are stored entirely in the browser:

- **Auto-save** — A single `AutosaveState` record keyed by `"current"` in IndexedDB, updated periodically. Contains `meta`, `blocks`, and `savedAt` timestamp.
- **Snapshots** — Named manual saves stored as individual records keyed by `id`. Up to 50 snapshots; oldest are auto-evicted when the cap is exceeded. Each snapshot preserves the full document state including embedded images.
- **Pord file format** — `.pord.json` files contain `{ format: "pord", version: 1, meta, blocks }` and are human-inspectable. Images remain as base64 strings within the blocks array.

> [!TIP]
> IndexedDB offers 50 MB to several GB of storage (browser-dependent), far exceeding localStorage's ~5 MB limit. This is essential for documents with multiple embedded images.

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 16 (App Router) + React 19 + TypeScript |
| Styling | Tailwind CSS 4 + custom CSS (A4 preview, RTL) |
| Components | shadcn/ui |
| Drag & Drop | @dnd-kit/core + @dnd-kit/sortable |
| Storage | IndexedDB (primary) + localStorage (fallback) |
| Typography | Vazirmatn (Persian font family — Regular, Medium, Bold) |
| Export | Self-contained HTML → browser print-to-PDF + Word-compatible `.doc` |
| Layout | Full RTL (`dir="rtl"`, `lang="fa"`) |

## Production Deployment

A `Caddyfile` is included for production with Caddy reverse proxy. Build for standalone:

```bash
bun run build
NODE_ENV=production bun .next/standalone/server.js
```
