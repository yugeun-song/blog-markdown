# blog-markdown

Posts for vmfault.dev, rendered by the private `blog` repo, which mounts this one at `blog/content/`. Layout, `meta.json` and deployment: [README.md](README.md). All post work happens here, never in `blog/content/`.

## Prose

- Tone: 평이체 (`~다`, `~이다`, `~한다`), the neutral register of papers and technical reports. Not 격식체 (`~습니다`), not 반말. State facts with `~이다` / `~한다`. Keep `~할 수 있다` / `~로 보인다` for real uncertainty. Identifiers, terms, English messages and quotes stay verbatim. Chat replies keep the global 격식체.
- 띄어쓰기: 조사는 앞말에 붙인다(한글 맞춤법 제41항). 앞말이 영어·코드·숫자여도 같다: `ftrace는`, `QEMU로`, `set_event`의, `nop`이. 서술격조사(`nop`인, `1`이면), 하다·되다 파생어(`emit한다`, `trace된다`), 접미사(`CPU당`, `Makefile들이`)도 붙인다. 뒤에 별도 명사·의존명사·부사가 오면 띄운다: `tracepoint 이벤트`, `Rust 등`, `cat 같은`. 코드 펜스 안에는 적용하지 않는다.
- `# Title` must equal `meta.json.title` character for character, because the page `<h1>` comes from `meta.json`. Do not add `readTime`.
- Images are centered by default. Add a lossless WebP twin with the engine's `build/encode-webp.py`.

## Code

- Every fence names a language. `text` covers kernel config, boot parameters, logs, pseudo-code and sysfs paths.
- UTF-8 with LF line endings. Non-C languages follow their idiomatic style (PEP 8, gofmt, rustfmt, common Bash).
- C/C++: pick a tier from the source, and keep one tier across a series.
  1. Linux kernel code, quoted or illustrative: kernel coding style exactly (tabs, K&R braces, function brace on its own line, `goto` cleanup). Never reformat it.
  2. Code the user supplied (their module or driver, or a cited third party): keep it exactly as given.
  3. Examples Claude writes, kernel-module-like ones included: 4-space indent and no tabs, braces on every control body, K&R braces for control flow, function brace on its own line.
- Kernel source in several versions comes from the engine's `resolve_kernel.py` into `source-snippets.json`, not by hand.

## Math

Inline `$…$`, display `$$…$$`, rendered at build time. Inside table cells write `\lvert` / `\rvert` instead of `|`.

## Diagrams

Structured data never goes into ASCII art. Use a table, Mermaid or a memory-layout SVG.

### Mermaid

Choose the type that avoids collisions by construction:

1. `sequenceDiagram` for ordered interactions: syscall traces, packet flow, handshakes, RPC, lifecycles.
2. `flowchart TD` / `LR` with one direction and no cycles for pipelines and decision trees.
3. `flowchart TD` with `subgraph` rank pinning for cycles and state machines, instead of `stateDiagram-v2`.

- A node label is by default a function name on one line, with the Korean explanation in the prose. Use a longer or Korean label only when nothing shorter is accurate, then check phone width in every theme. Long labels break phone layouts.
- Edge labels sit on the arrow. Keep them to 1–3 words. Their background is transparent.
- Node classes: `:::accent` (gold), `:::info` (green), `:::warn` (orange), `:::danger` (red), `:::muted` (gray, dashed), `:::code`.
- Node labels use the body font. Mark a node `:::code` when its whole label is a code token (signature, variable, flag, hex value), and it switches to the code font. HTML labels are off, so one node has one font.
- Prefer one language per diagram.

### Memory tables

For wide bitfields, struct layouts and memory regions, write a raw `<table class="mem-layout">`. Use `<th>` for headers, `<td class="field">` for data, `class="pad"` for padding and `class="offset"` for offsets. Use `colspan` for multi-byte or multi-bit fields. The engine draws the table as SVG with click-to-expand. 32-column bit tables wrap text vertically when narrow.

### Memory-layout SVG

Draw address spaces, stack frames, pointer chains and region layouts this way when the point is addresses, stored values and the pointers between them. Write a raw `<svg class="mem-diagram">` in `index.md`. The engine wraps it in `<figure class="mem-diagram-wrap">`, which has the code-block background and border, `var(--wrap-radius)` and click-to-expand. The two diagrams in `gdb-kernel-debugging/index.md` are the reference.

- Canvas: `viewBox="0 0 700 H"` and `font-family="code-mono, monospace"` on the `<svg>`, with no `width` or `height`. The wrapper scales the diagram to the column, up to 700px.
- Orientation: the high address is at the top. The axis sits at x=32: a line with a filled triangle pointing up, bold `high` above it, bold `low` below it.
- Column: left rail x=245, right rail x=455 (210 wide), capped top and bottom. Horizontal dividers split it into regions, with the highest region on top. Region content is centered at x=350. Boundary addresses end at x=231.
- Colors come from CSS variables only, never hex:
  - frame, dividers, axis, ticks, arrows, arrowheads, arrow labels, default text: `var(--diagram-ink)`
  - real regions: one opaque `var(--diagram-area)` for all of them
  - omitted spans: opaque `var(--diagram-gap)`
  - register or pointer markers such as `X29 = sp`: `var(--syntax-keyword)`
  - the `⋮` glyph: `var(--text-secondary)`
  - Keep both fills opaque, with no `fill-opacity`. The diagram then looks the same inline over `--code-bg` and in the modal over `--diagram-container-bg`.
- Strokes: every line uses `style="…;stroke-width:var(--diagram-stroke)"`. The engine's `styles/base.css` sets the width once (2.4). Never put a `stroke-width` attribute on a single line.
- Text:

  | Element | Attributes |
  |---|---|
  | boundary address, left of the column | `font-size="15" font-weight="700" text-anchor="end"` at x=231, baseline on the boundary |
  | register marker | the same, red, just above its address |
  | region value | `font-size="15" font-weight="700" text-anchor="middle"` at x=350 |
  | region sub-label such as `(caller frame)` | `font-size="11"`, regular, centered about 24px below the value |
  | region without a value such as `truncated` | a `font-size="14"` word plus the 11px sub-label |
  | `⋮` in an omitted span | `font-size="26" font-weight="700"`, centered |
  | arrow label | `font-size="15"`, bold, `text-anchor="start"` right of the arrow's vertical leg. The text is the concrete `*(<address>)`: the stored pointer, which equals the target's start address, e.g. `*(0xffff800083fcbc90)`. |

- Reference arrows follow these rules without exception:
  - The tail starts at the vertical center of the source region, on the right rail (x=455).
  - The head lands on the target's lowest address, its bottom edge in this layout. For `*(ptr + offset)` it lands `offset` into the region, in proportion.
  - The route is a right-angle elbow outside the column to the right: horizontal, vertical, horizontal. Corners are rounded with `Q` at radius ~12. Arrows that share the margin take increasing x for their vertical legs. A leg moves inward when its label would overflow the canvas.
  - The arrowhead is an explicit triangle, `<path d="M … Z" style="fill:var(--diagram-ink)"/>`, never a `<marker>`, whose `var()` fill does not resolve.
  - The line runs a couple of units past the arrowhead's base, because butt caps otherwise leave a seam. The same holds for the axis.
- Scaling and theming come from the wrapper and the variables. Still check phone width and every theme.
- This section is the single source of truth. Update it whenever the design changes.
