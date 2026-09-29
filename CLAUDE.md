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

Structured data never goes into ASCII art. Use a table, Mermaid or a memory diagram. Every diagram follows the memory-layout figure in `gdb-kernel-debugging/index.md`: ink strokes, opaque area and gap fills, bold code-font labels, triangle arrowheads and radius-12 bends. The generators and their prompts live in the diagram-design.md repo (the engine's `diagrams/` submodule).

### Mermaid

Choose the type that avoids collisions by construction:

1. `sequenceDiagram` for ordered interactions: syscall traces, packet flow, handshakes, RPC, lifecycles.
2. `flowchart TD` / `LR` with one direction and no cycles for pipelines and decision trees.
3. `flowchart TD` with `subgraph` rank pinning for cycles and state machines, instead of `stateDiagram-v2`.

- A node label is by default a function name on one line, with the Korean explanation in the prose. Use a longer or Korean label only when nothing shorter is accurate, then check phone width in every theme. Long labels break phone layouts.
- Edge labels sit on the arrow. Keep them to 1–3 words. `-->` is the main path; `-.->` draws a dashed edge for an optional or asynchronous one.
- Node classes mark roles, one per node: `:::accent` the result or the point, `:::muted` something off the main path, `:::danger` an error. Every label uses the code font.
- Add no `%%{init}%%`, `classDef` or `style` lines. Prefer one language per diagram.

### Memory tables

For struct layouts, packet headers and register bitfields, write a ` ```memory-table ` fence with one JSON object: `unit` (`"byte"` or `"bit"`), `fields` in order from the lowest offset (`["name (type)", size]`, `{"pad": n}`, `{"other": "text", "size": n}`), and optional `cols`, `order`, `base`, `label`. The build computes the rows, spans and offsets and draws the table as a memory diagram. Raw `<table class="mem-layout">` still works.

### Memory layouts

For address spaces, stack frames, pointer chains and region layouts, write a ` ```memory-layout ` fence with one JSON object: `label` (the aria-label) and `regions` from the highest address to the lowest. A region has one of `value`, `word` or `"gap": true`, and optionally `sub`, `start` (its lowest address), `marker`, `id`, `to` (the region its value points into) and `h`. The build draws the column, addresses, arrows and `*(value)` labels, and stops on an invalid spec. The two fences in `gdb-kernel-debugging/index.md` are the reference. Check phone width and every theme.
