# blog-markdown

Content repository for the `vmfault` blog. Mounted as a git submodule at `blog/content/`. Pushing to `main` triggers `.github/workflows/notify.yml`, which checks out the sister `blog` repo, runs `git submodule update --remote content`, commits the bumped pointer, and pushes — that push triggers Cloudflare Pages deploy.

Sister repo: `yugeun-song/blog` (private; build/render code).

## Status

`main` is post-reset. Commit `02ff209` ("AI로 생성한 목업 콘텐츠 전체 삭제, 초기 상태로 리셋") cleared all AI-generated mock content. New posts must be authored fresh. The 11 mock posts visible inside the sister repo's `blog/content/` checkout come from a different snapshot — kept as mock references for tooling and design only, not authoritative.

## Post layout

Each post = one slug directory:

- `{slug}/meta.json` — metadata
- `{slug}/index.md` — body (no frontmatter; first line is `# Title`)
- `{slug}/<assets>` — optional per-post images and supporting files. Page-bundle pattern: place assets directly inside the slug directory (flat or under `images/`) and reference from `index.md` via relative path (`./foo.png` / `./images/foo.png`). The build copies every entry except `meta.json` and `index.md` to `dist/posts/{slug}/` verbatim, preserving subfolder layout. Final URL: `/posts/{slug}/foo.png`. Rendered images are center-aligned by default via the global `img` rule in sister `blog/styles/base.css` (`display: block; margin-inline: auto`); override per-image with explicit inline styles or wrapping HTML only when a different alignment is intentional.

`{slug}` is kebab-case ASCII; the directory name is the URL path verbatim. Example: `cfs-scheduler/` → `https://<deploy-url>/posts/cfs-scheduler/`.

## meta.json schema

| Field | Type | Required | Notes |
|---|---|---|---|
| `title` | string | yes | Page title; matches first line of `index.md` |
| `date` | string | yes | `YYYY-MM-DD`; sort key |
| `tags` | string[] | yes | Lowercase ASCII recommended |
| `excerpt` | string | yes | Card preview, search snippet |
| `series` | string | no | `category/subcategory` (e.g. `linux-kernel/process`) |
| `seriesOrder` | number | no | 1-based topical order, not chronological |

## Body rules

- Tone: **평이체** (`~다 / ~이다 / ~한다`). Not 격식체 (`~습니다`), not 반말 (`~야`). The register is the objective, neutral tone of academic papers and technical reports.
- 띄어쓰기 — **조사는 앞말에 붙인다** (국립국어원 한글 맞춤법 §41). 앞말이 영어·코드·숫자여도 동일: `ftrace는`, `tracefs에서`, `QEMU로`, `configuration을`, 인라인 코드 뒤도 `set_event`의 / `nop`이. 서술격조사(`nop`인, `1`이면), 명사+하다/되다 파생(`emit한다`, `trace된다`), 접미사(`CPU당`, `Makefile들이`)도 붙인다. 단 뒤가 **별도 명사·의존명사·부사**면 띄운다: `tracepoint 이벤트`, `Kernel hacking 항목`, `Rust 등`, `tracer 중`, `cat 같은`.
- First line is `# Title`. Must match `meta.json.title` exactly.
- Code fences must declare a language (`c`, `python`, `bash`, `text`, …). Use `text` for config dumps and pseudo-code.
- GFM extensions allowed: tables, strikethrough, task lists.
- Math (KaTeX): inline `$...$`, display `$$...$$`. Build-time SSR — no client JS for math. Inside markdown table cells, use `\lvert` / `\rvert` instead of `|` to avoid colliding with the table parser.
- Diagrams (Mermaid): ` ```mermaid ` fence. Renders client-side; theme follows the active site theme. Available node classes: `:::accent`, `:::info`, `:::warn`, `:::danger`, `:::muted`. Keep edge labels short (`SYN+ACK / ACK`, `2MSL timeout`, `wakeup`) — long labels are harder for auto-layout to place.

  **Diagram type selection priority** — pick the type that minimizes label/arrow collisions by construction, not the one that "looks right":

  1. **`sequenceDiagram`** — first choice for any sequential interaction (syscall trace, packet flow, handshake, RPC, lifecycle). Grid layout (participants on fixed X axes, time-ordered Y) is collision-free by construction. The diagram in `iommu-internals/index.md` is the reference for "clean by default".
  2. **`flowchart TD/LR`** with single direction and no cycles — second choice for unidirectional flows (pipelines, decision trees, request paths).
  3. **`stateDiagram-v2`** or `flowchart` with cycles/hubs — last resort. Auto-layout cannot avoid label-on-node and label-on-edge collisions in graphs with cycles, hub nodes, or many same-named labels (e.g. `ACK recv` repeated). When unavoidable, a global CSS `paint-order: stroke` rule in the sister `blog` repo (`styles/components.css`) draws an opaque outline around edge label glyphs in the mermaid container background color, keeping labels readable on top of arrows. This is a visual patch, not a layout fix — pick (1) or (2) whenever possible.
- Wide tabular layouts (bitfields, struct layouts, memory regions): write raw HTML `<table class="mem-layout">` directly. Cells use `field` (data), `pad` (padding), `offset` (offset column); headers use `<th>`. Multi-byte fields: `colspan`. The build wraps it in `<div class="table-scroll">`; client JS converts it to a responsive inline SVG with click-to-expand. Use this pattern for horizontal data instead of forcing wide mermaid fan-outs — saves vertical space.

## Series naming

- Format: `category/subcategory` (lowercase, hyphens, `/` as hierarchy separator).
- URL slug = `replace("/", "-")` at build time. Display name keeps `/`.
- Order is topical (prerequisite first), not chronological.
- Examples: `linux-kernel/process`, `linux-system-programming/io`, `math/information-theory`. Single-tier (`bash`) is allowed.
- Tags and series are independent namespaces.

## Branching

Single-branch (`main`). Direct commits, no feature branches. Roll back via `git revert` or `git restore`.

## Auto-deploy

`.github/workflows/notify.yml` watches `main` push events:

1. Checks out `yugeun-song/blog` with `BLOG_REPO_TOKEN`.
2. `git submodule update --remote content`.
3. Commits "Update content submodule" if there's a diff and pushes.
4. The pushed commit triggers Cloudflare Pages.
