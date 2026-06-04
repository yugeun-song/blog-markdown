# blog-markdown

Content repository for the [vmfault](https://github.com/yugeun-song/blog) blog. Mounted as a git submodule at the main blog project's `content/` path. Pushing to `main` triggers GitHub Actions to bump the blog repo's submodule pointer, which in turn triggers a Cloudflare Pages build.

## Status

`main` is post-reset (commit `15f6935` cleared all AI-generated mock content). New posts must be authored fresh; the mock posts seen in some local checkouts are kept only as references for tooling and design experiments.

## Directory layout

Each post is one slug directory:

- `{slug}/meta.json` — metadata
- `{slug}/index.md` — markdown body (no frontmatter; first line is `# Title`)
- `{slug}/<assets>` — optional per-post images and supporting files. Co-located with the markdown (flat in the slug directory, or under a subfolder like `images/`). Reference from `index.md` via relative path (`./foo.png` or `./images/foo.png`). The build copies every entry except `meta.json` and `index.md` verbatim to `dist/posts/{slug}/`, preserving subfolder layout. Rendered images are center-aligned by default via the global `img` rule in the sister `blog/styles/base.css`.

The directory name becomes the URL path verbatim: `rust-async-runtime/` → `https://blog-213.pages.dev/posts/rust-async-runtime/`.

Slugs are kebab-case ASCII only.

## `meta.json` schema

```json
{
  "title": "CFS Scheduler Internals",
  "date": "2026-03-01",
  "tags": ["linux", "kernel", "scheduler"],
  "excerpt": "One-line summary",
  "series": "linux-kernel/process",
  "seriesOrder": 3
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `title` | string | yes | Page title, card title, `<title>` |
| `date` | string | yes | `YYYY-MM-DD`; sort key |
| `tags` | string[] | yes | Filter chips and tag cloud; lowercase ASCII recommended |
| `excerpt` | string | yes | Card preview, search result snippet |
| `series` | string | no | `category/subcategory` form. Display keeps `/`; URL slug replaces it with `-` |
| `seriesOrder` | number | no | Topical (prerequisites first), 1-based, not chronological |

## Series naming

Lowercase, hyphens, `/` as the hierarchy separator. The build replaces `/` with `-` for the URL slug.

- `linux-kernel/net`
- `linux-kernel/process`
- `linux-system-programming/io`
- `math/information-theory`
- `math/optimization`
- `bash` (single-tier is allowed)

Tags and series are independent namespaces.

## `index.md` rules

- No frontmatter — `meta.json` owns the metadata
- First line is `# Title` and must match `meta.json.title`
- `h2` / `h3` are auto-registered in the TOC at build time (`rehype-slug` assigns ids)
- Every code fence must declare a language (`c`, `python`, `bash`, `text`, …); use `text` for config dumps and pseudo-code
- GFM extensions: tables, strikethrough, task lists
- Tone: 평이체 (`~다 / ~이다 / ~한다`). Not 격식체 (`~습니다`), not 반말. The register is the objective, neutral tone of academic papers and technical reports.

### Math (KaTeX)

Inline `$...$`, display `$$...$$`. Built at SSR time, so no client JS is shipped for math. Inside markdown table cells, use `\lvert` / `\rvert` instead of `|` to avoid colliding with the table parser.

### Diagrams (Mermaid)

Use ` ```mermaid ` fences. Renders client-side; theme follows the active site theme. The blog renders flowcharts with the ELK layout engine for clean spacing — keep edge labels short anyway, since long labels still collide with nodes. Available node classes: `:::accent`, `:::info`, `:::warn`, `:::danger`, `:::muted`.

For sequential interactions (syscall traces, packet flow, handshake), prefer `sequenceDiagram` — its grid layout is collision-free by construction. State machines (`stateDiagram-v2`) share dagre's auto-layout limits and benefit from being rewritten as flowcharts.

### Wide tabular layouts (`<table class="mem-layout">`)

For bitfields, struct layouts, and memory regions, write raw HTML `<table class="mem-layout">` directly inside markdown. The build pipeline wraps it in `<div class="table-scroll">`; client JS converts it to a responsive inline SVG with click-to-expand.

- Cell classes: `field` (data), `pad` (padding bytes), `offset` (offset column); headers are `<th>`
- `colspan` for multi-byte / multi-bit fields
- Click opens the expand modal — zoom, pan, reset

Use this pattern for wide horizontal data instead of forcing a wide mermaid fan-out.

## Branch policy

Single branch, `main`. No feature branches; commit and push directly. Roll back with `git revert` or `git restore`.

## Auto-deploy workflow

`.github/workflows/notify.yml` reacts to `main` push events. It checks out `yugeun-song/blog` and runs `git submodule update --remote content`. If there is a diff, it pushes an `Update content submodule` commit, and that commit triggers Cloudflare Pages.
