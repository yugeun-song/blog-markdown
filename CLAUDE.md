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
- Edges render solid. The engine draws `-.->` solid too, so write `-->` and `-- label -->`.
- Node classes mark roles, and each theme colors them: `:::accent` the result or the point, `:::info` the normal or fast path, `:::warn` a slow or risky step, `:::danger` an error, `:::muted` something off the main path (dashed outline).
- Node labels use the body font. `:::code` switches a node to the code font when its whole label is a code token (signature, variable, flag, hex value). A bare function name is not a code token. A node takes one class, so a code token that fits a role takes the role class. A label never mixes fonts.
- Prefer one language per diagram.

### Memory tables

For wide bitfields, struct layouts and memory regions, write a raw `<table class="mem-layout">`. Use `<th>` for headers, `<td class="field">` for data, `class="pad"` for padding and `class="offset"` for offsets. Use `colspan` for multi-byte or multi-bit fields. The engine draws the table as SVG with click-to-expand. 32-column bit tables wrap text vertically when narrow.

### Memory-layout SVG

Draw address spaces, stack frames, pointer chains and region layouts this way when the point is addresses, stored values and the pointers between them. Write a raw `<svg class="mem-diagram">` in `index.md`. The engine wraps it in `<figure class="mem-diagram-wrap">`, which has the code-block background and border, `var(--wrap-radius)` and click-to-expand. The two diagrams in `gdb-kernel-debugging/index.md` are the reference.

- Canvas: `viewBox="0 0 700 H"` and `font-family="code-mono, monospace"` on the `<svg>`, with no `width` or `height`. H is about 40 below the column's bottom. The wrapper scales the diagram to the column, up to 700px.
- Axis: the high address is at the top. At x=32 a line runs from the column's bottom up to y=82, under the triangle `M 32 74 L 26 86 L 38 86 Z`. `high` sits at y=54 and `low` 24 below the column's bottom, both bold 15 and centered on x=32.
- Column: left rail x=245, right rail x=455 (210 wide), top at y=72, capped top and bottom. Horizontal dividers split it into regions, with the highest region on top. Region content is centered at x=350. Each labeled boundary gets a 12-unit tick (x 233–245) and an address ending at x=231.
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
  | boundary address, left of the column | `font-size="15" font-weight="700" text-anchor="end"` at x=231, baseline 4 below the boundary |
  | register marker | the same in `var(--syntax-keyword)`, baseline 12 above the boundary; its address then moves to 6 below |
  | region value | `font-size="15" font-weight="700" text-anchor="middle"` at x=350, baseline 4 above the region's center |
  | region sub-label such as `(caller frame)` | `font-size="11"`, regular, centered about 24 below the value |
  | region without a value such as `truncated` | a `font-size="14"` word plus the 11px sub-label |
  | `⋮` in an omitted span | `font-size="26" font-weight="700"`, baseline about 10 below the span's center |
  | arrow label | `font-size="15"`, bold, `text-anchor="start"` 8 right of the vertical leg, baseline near the leg's midpoint. The text is the concrete `*(<address>)`: the stored pointer, which equals the target's start address, e.g. `*(0xffff800083fcbc90)`. |

- Reference arrows follow these rules without exception:
  - The tail starts at the vertical center of the source region, on the right rail (x=455).
  - The head lands on the target's lowest address, its bottom edge in this layout. For `*(ptr + offset)` it lands `offset` into the region, in proportion.
  - The route runs right 26, turns through a radius-12 `Q` corner, runs a vertical leg at x=493 and comes back the same way: `M 455 ys H 481 Q 493 ys 493 ys∓12 V yt±12 Q 493 yt 481 yt H 464`, with ys the tail, yt the head and the upper signs for an upward arrow. Arrows whose vertical spans overlap take increasing x for their legs. A leg moves inward when its label would overflow the canvas.
  - The arrowhead is its own triangle, `M 454 yt L 466 yt-6 L 466 yt+6 Z` with `fill:var(--diagram-ink)`, never a `<marker>`, so its size and position stay explicit.
  - The line ends at x=464, 2 past the arrowhead's base, because butt caps otherwise leave a seam. The axis line overlaps its triangle's base by 4.
- Scaling and theming come from the wrapper and the variables. Still check phone width and every theme.
- This section is the single source of truth. Update it whenever the design changes.
