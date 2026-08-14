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
- Diagrams (Mermaid): ` ```mermaid ` fence. Renders client-side; theme follows the active site theme. Available node classes: `:::accent`, `:::info`, `:::warn`, `:::danger`, `:::muted`. Default to short edge labels (`SYN+ACK / ACK`, `2MSL timeout`, `wakeup`), since long ones are harder for auto-layout to place. Same for keeping Korean out of node labels. These are defaults; use a longer or Korean label when it is the only accurate one, then verify the render at phone width and in every theme.

  **Diagram type selection priority** — pick the type that minimizes label/arrow collisions by construction, not the one that "looks right":

  1. **`sequenceDiagram`** — first choice for any sequential interaction (syscall trace, packet flow, handshake, RPC, lifecycle). Grid layout (participants on fixed X axes, time-ordered Y) is collision-free by construction. The diagram in `iommu-internals/index.md` is the reference for "clean by default".
  2. **`flowchart TD/LR`** with single direction and no cycles — second choice for unidirectional flows (pipelines, decision trees, request paths).
  3. **Cyclic graphs and state machines**: `flowchart TD` with `subgraph` blocks pinning the ranks, not `stateDiagram-v2`. Auto-layout routes back-edges around the whole figure and strands their labels far from the arrow; pinning ranks is what shortens those edges. For what is left, a global CSS `paint-order: stroke` rule in the sister `blog` repo (`styles/components.css`) outlines edge-label glyphs so they stay readable over arrows. That is a visual patch, not a layout fix.
- Wide tabular layouts (bitfields, struct layouts, memory regions): write raw HTML `<table class="mem-layout">` directly. Cells use `field` (data), `pad` (padding), `offset` (offset column); headers use `<th>`. Multi-byte fields: `colspan`. The build wraps it in `<div class="table-scroll">`; client JS converts it to a responsive inline SVG with click-to-expand. Use this pattern for horizontal data instead of forcing wide mermaid fan-outs — saves vertical space.

## Memory-layout diagrams (inline SVG)

Use these for address spaces, stack frames, pointer chains, and struct/region layouts — anywhere the point is **the addresses, the value stored at each address, and the reference (pointer) relationships between them**. Author each one as a raw `<svg class="mem-diagram">…</svg>` block placed directly in `index.md`. The build extracts the block, wraps it in `<figure class="mem-diagram-wrap">` (a padded container whose background and border match the code / shell-output blocks — `var(--code-bg)` / `var(--code-border)` — rounded with the shared `var(--wrap-radius)`, plus a click-to-expand modal), and makes it responsive. Color every element with CSS variables only — never hardcode hex — so all themes render correctly.

Follow this spec exactly. The two diagrams in `gdb-kernel-debugging/index.md` are the reference; copy their structure.

**Canvas.** Use `<svg viewBox="0 0 700 H">` with matching `width`/`height` attributes; `H` is whatever the content needs. Set `font-family="Cascadia Code, monospace"` on the `<svg>` so all text inherits it.

**Orientation.** Put the high address at the TOP and the low address at the BOTTOM. Draw a vertical axis on the far left (x=32): a line with an explicit triangle arrowhead pointing up, a `high` label above it, a `low` label below. Make the `high` / `low` labels bold (`font-weight="700"`).

**The memory column.** Draw one vertical column, horizontally centered: left rail at x=245, right rail at x=455 (210 wide), capped top and bottom. Split it into stacked regions with horizontal dividers (highest region on top, lowest at the bottom). Center region content at x=350; right-anchor the left-side address labels at x=231.

**Colors (CSS variables only).**
- Frame, dividers, axis line, axis arrowhead, address tick marks, and all default text → `var(--text-primary)`.
- Occupied (real) partition fill → `var(--diagram-area)` — an OPAQUE per-theme solid (light mint in clean-light, a clean forest/teal green in the dark themes). Tune it per theme to fit that theme's canvas and mood; use one unified fill for every real region (no per-region type colors unless a post specifically needs that). Keep it opaque (see consistency note).
- Omitted / gap bands (the `⋮` spans) fill → `var(--diagram-gap)` — an OPAQUE per-theme solid: a neutral just off the canvas (light gray in clean-light, dark gray in midnight, navy in spaceduck).
- Reference arrows, their arrowheads, and their labels → `var(--text-primary)` (black, same as the frame).
- Register / pointer markers such as `X29 = sp` → `var(--syntax-keyword)` (red).
- The `⋮` glyph → `var(--text-secondary)` (deep gray).

**Inline vs expanded consistency.** Keep `--diagram-area` and `--diagram-gap` OPAQUE. Opaque fills are background-independent, so the diagram looks identical inline and when expanded in the modal — only the outer margin background differs. (A semi-transparent fill would composite over the inline wrapper `var(--code-bg)` versus the modal `var(--diagram-container-bg)` and look subtly different once expanded — so never put `fill-opacity` on the region fills.)

**Stroke width (global, uniform).** Every stroke — outer frame, caps, partition dividers, axis line, address ticks, and reference arrows — uses ONE shared width via `style="…;stroke-width:var(--diagram-stroke)"` (do not set per-line `stroke-width` attributes). `--diagram-stroke` (currently `2.4`) is defined once in `:root` (sister `blog` repo, `styles/base.css`), alongside `--wrap-radius`; changing it there rescales every diagram's lines at once. All lines stay the same thickness.

**Text — weight, size, position, alignment.**
- Boundary address labels (left of the column, one per region boundary): `font-size="15" font-weight="700" text-anchor="end"` at x=231, baseline on the boundary. Always bold. (These side labels read larger than the in-region sub-labels.)
- Register / pointer marker (`X29 = sp`): `font-size="15" font-weight="700" text-anchor="end"`, red, sitting just above its address label.
- Region value (the hex address or instruction stored in the region): `font-size="15" font-weight="700" text-anchor="middle"` at x=350, `var(--text-primary)`. This is the most prominent token in the region — bold, but one notch smaller than a heading.
- Region sub-label (`(caller frame)`, `(.text)`, `(kernel stack)`): `font-size="11"`, regular weight, centered, about 24px below the value.
- A region with no concrete value (e.g. `truncated`) uses a `font-size="14"` centered word plus the `font-size="11"` sub-label.
- `⋮` (omitted span): `font-size="26" font-weight="700"`, `var(--text-secondary)`, centered in the gap band.
- Reference-arrow label: `font-size="15"`, bold, black (`var(--text-primary)`), left-anchored (`text-anchor="start"`) just to the right of the arrow's vertical leg, vertically near its arrow. Write the concrete dereference `*(<address>)` — the source's stored pointer value, which equals the destination's start address (e.g. `*(0xffff800083fcbc90)`), not a generic `*(void **)`. Pull the arrow's vertical leg in toward the column far enough that the address label fits inside the canvas without overflowing.

**Reference arrows — follow this rule without exception.**
- Start the tail at the CENTER of the source region (the midpoint of its address span), on the right rail (x=455).
- Land the head on the DESTINATION region's LOWEST (start) address edge — its bottom edge in this high-at-top layout — never its center or an arbitrary point. The machine accesses an object starting from its lowest address, so the pointer lands at the start.
- For `*(some_ptr + offset)`, land the head `offset` worth of distance into the target (proportional to where `start + offset` falls within the region), not at the start.
- Route the arrow as a right-angle elbow OUTSIDE the column, to the right: go horizontal from the source center, then vertical, then horizontal back to the destination edge. Round the corners with quadratic curves (`Q`, radius ~12). When several arrows share the right margin, push each one's vertical leg to a larger x so they never overlap.
- Draw the arrowhead as an explicit filled triangle (`<path d="M … Z" style="fill:var(--text-primary)"/>`). Do NOT use an SVG `<marker>` — a `var(--…)` fill inside a `<marker>` does not resolve in browsers.

**Responsiveness and themes.** Nothing extra is required: the wrapper's `max-width:100%; height:auto` scales the SVG to the container on narrow screens, and the `var(--…)` colors retheme automatically. Still confirm a new diagram reads on a phone-width screen and in every theme.

**Keep this spec current.** Whenever the diagram design changes, update this section so it stays the single source of truth.

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
