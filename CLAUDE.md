# blog-markdown

Posts for vmfault.dev, rendered by the private `blog` repo, which mounts this one at `blog/content/`. Layout, `meta.json` and deployment: [README.md](README.md). All post work happens here, never in `blog/content/`.

## Prose

- Tone: 평이체 (`~다`, `~이다`, `~한다`), the neutral register of papers and technical reports. Not 격식체 (`~습니다`), not 반말. State facts with `~이다` / `~한다`. Keep `~할 수 있다` / `~로 보인다` for real uncertainty. Identifiers, terms, English messages and quotes stay verbatim. Chat replies keep the global 격식체.
- 띄어쓰기: 조사는 앞말에 붙인다(한글 맞춤법 제41항). 앞말이 영어·코드·숫자여도 같다: `ftrace는`, `QEMU로`, `set_event`의, `nop`이. 서술격조사(`nop`인, `1`이면), 하다·되다 파생어(`emit한다`, `trace된다`), 접미사(`CPU당`, `Makefile들이`)도 붙인다. 뒤에 별도 명사·의존명사·부사가 오면 띄운다: `tracepoint 이벤트`, `Rust 등`, `cat 같은`. 코드 펜스 안에는 적용하지 않는다.
- Paragraphs: one meaningful unit of context per paragraph, as in the ftrace and gdb posts, usually three to six sentences. A claim, its reasons and its example stay together, and a sentence that introduces a code block, list or figure ends the paragraph it belongs to. Never split a unit with blank lines. A paragraph stands alone short only as a section opener, between two code blocks, or for parallel cases.
- When editing a post, act like a formatter: fix grammar and spacing, and regroup the paragraphs this way, in every post alike.
- `# Title` must equal `meta.json.title` character for character, because the page `<h1>` comes from `meta.json`. Do not add `readTime`.
- For now every page shows the engine's `static/og-default.png`, because its `POST_OG_IMAGES` is off, so a post needs no `og.png`. With the flag on, a new post, or a change to its title, date or tags, needs a new link preview: run `npm run og -- ../blog-markdown/<slug>` in the engine after a build or preview, since it reads the built CSS, and commit the `og.png` it writes. Until then the post shows the site's default image.
- Images are centered by default. Add a lossless WebP twin with the engine's `build/encode-webp.py`.

## Code

- Every fence names a language. `text` covers kernel config, boot parameters, logs, pseudo-code and sysfs paths.
- UTF-8 with LF line endings. Non-C languages follow their idiomatic style (PEP 8, gofmt, rustfmt, common Bash).
- C/C++: pick a tier from the source, and keep one tier across a series.
  1. Linux kernel code, quoted or illustrative: kernel coding style exactly (tabs, K&R braces, function brace on its own line, `goto` cleanup). Never reformat it.
  2. Code the user supplied (their module or driver, or a cited third party): keep it exactly as given.
  3. Examples Claude writes, kernel-module-like ones included: 4-space indent and no tabs, braces on every control body, K&R braces for control flow, function brace on its own line.
- Kernel source in several versions comes from the engine's `resolve_kernel.py` into `source-snippets.json`, not by hand. Cite each version at `latest`, its newest stable tag. The tool stores the full commit hash, and the page shows the version as `v6.12` and the first 6 characters of the hash, linked to git.kernel.org.
- Code from any other source, an open git repository or the user's own file, takes an inline `<!-- source-ref -->` marker over its fence with only the attributes it has (`repo`, `project`, `path`, `version`, `commit`, `start`), as the glibc quote in `ethernet-header/index.md` does. The header shows only those parts, and `header` picks them or turns the bar `off`. The engine's `docs/source-excerpts.md` lists every attribute and forge.
- Describe kernel behavior in terms that hold on every architecture, and quote code without architecture- or endian-specific branches where a portable part says the same.

## References

- Every post that cites sources ends with them, written as GFM footnotes: a marker `[^n]` right after the claim, before the sentence's period, and the definition `[^n]: ...` at the end of `index.md`, in number order. The label is the reference's number in order of first citation, so the first source cited is `[^1]`; the build fails on a label out of that order, so renumber every label after inserting or moving a citation. Write a `## References` heading right before the definitions. The build shows each marker as a clickable `[n]` (`#ref-n`) and lists the entries under that heading, which takes no section number, each opening with a plain `[n]`; it fails when a post with footnotes has no heading right before them.
- One entry per work, as in IEEE style, and a citation is always a bare `[^n]`. The place inside the work goes in the prose, in parentheses next to the claim and linked to that place: a section as `([RFC 9542 3절](https://www.rfc-editor.org/rfc/rfc9542#section-3))`, source code as its file and line `path:line`, linked to that line of the cited commit on git.kernel.org (`#nLINE`), then the function, structure or macro when the sentence does not name it, as in `리눅스 브리지([`net/bridge/br_input.c:347`](…#n347), `br_handle_frame()`)는`. Places from two sources in one spot share the parentheses, separated by a semicolon. Check every link in Chrome.
- Cite a source only for what was actually taken from it, after reading the passage. A plausible source for a claim is not a citation. A claim no read source states is restated, cut, or left uncited, never given a borrowed reference.
- Primary sources only: standards and registries (IEEE, IETF RFCs, IEEE RA listings), the kernel at the newest v6.18 tag, glibc and other project sources. Never blogs, wikis, forums or Q&A sites. When a standard cannot be read, verify the claim in equivalent professional sources, such as its working group's public drafts and revision requests, its standards body's interpretations and the registries, and cite those; tell the user when none states it.
- Entries follow IEEE style: authors or organization, the title in quotes and linked to the source, document number and year. A kernel or glibc entry is the release, `commit` and its first 12 hexadecimal digits, linked to the source tree at that commit; the files go in the citations.
- Every entry links to its source, and the link must open in a real browser; check them in Chrome with a regular user agent. Only works as well known as "Attention Is All You Need" may go without a link.

## Captions

The build puts a numbered caption under each object, counted per kind in document order: 인용 for `source-ref` excerpts, 예제 for code fences, 사진 for an image alone in its paragraph, 표 for tables and ` ```text-table ` figures, 다이어그램 for Mermaid, memory and struct-chain diagrams and raw `<svg>`. Shell, log and `text` fences get no number. To change a kind, put `<!-- caption kind="graph" -->` on the line right before the object; the kinds are `citation`, `example`, `photo`, `graph`, `table`, `diagram` and `none`. Writing an object's caption in the prose, as in 다이어그램 2는 or 표 1과, links the mention to that object. A counter after the number (표 2개, 사진 3장) keeps it plain text. The build cannot tell when an object added above shifts the numbers, so recheck every mention after inserting one.

## Math

Inline `$…$`, display `$$…$$`, rendered at build time. Inside table cells write `\lvert` / `\rvert` instead of `|`.

## Diagrams

Structured data never goes into ASCII art. Use a table, Mermaid or a memory diagram. Every diagram follows the memory-layout figure in `gdb-kernel-debugging/index.md`: ink strokes, opaque area and gap fills, bold code-font labels in one style inside every box (a second line keeps the size and color), boxes about 2.3 times as wide as tall with the longest label filling about 60 to 70 % of the width, triangle arrowheads on pointers, open arrowheads on dimension lines, radius-12 bends on arrows while boxes keep square corners, and dashes for partial or virtual regions. Labels render at the size of code text. Strokes join without seams, notches or hairline gaps, and where a mark such as a bracket meets a reference line such as an extension line, the reference line stays in front; the kit's `lintJoins` checks the JSON forms. The generators and their prompts live in the diagram-design.md repo (the engine's `diagrams/` submodule).

Color comes from one rainbow of pale washes, kept for diagrams and tables: `red` (a crimson), `orange`, `yellow`, `green`, `blue`, `indigo` and `purple`, all at one lightness and chroma per theme. A tone fill such as `"fill": "purple"` draws its wash; the hex values per theme are under "Rainbow washes" in the engine's `docs/palette.md`. Give a tone only to what the figure is about.

### Mermaid

Choose the type that avoids collisions by construction:

1. `sequenceDiagram` for ordered interactions: syscall traces, packet flow, handshakes, RPC, lifecycles.
2. `flowchart TD` / `LR` with one direction and no cycles for pipelines and decision trees.
3. `flowchart TD` with `subgraph` rank pinning for cycles and state machines, instead of `stateDiagram-v2`.

- A node label is by default a function name on one line, with the Korean explanation in the prose. Use a longer or Korean label only when nothing shorter is accurate, then check phone width in every theme. Long labels break phone layouts.
- Edge labels sit on the arrow. Keep them to 1–3 words. `-->` is the main path; `-.->` draws a dashed edge for an optional or asynchronous one.
- The renderer routes every flowchart edge: straight when the two boxes line up, otherwise one bend between ranks into a third of the box. Do not steer edges with invisible links or spacer nodes.
- Node classes mark roles, one per node: `:::accent` the result or the point, `:::muted` something off the main path, `:::danger` an error. Every label uses the code font.
- Add no `%%{init}%%`, `classDef` or `style` lines. Prefer one language per diagram.
- The engine draws every figure before deployment, so the page runs no Mermaid. After adding or changing a fence, run `npm run mermaid -- ../blog-markdown/<slug>` in the engine, after any build or preview since it reads the built CSS, and commit the `mermaid.json` it writes beside `index.md` with the post. The Cloudflare build cannot draw and fails on a figure missing from that file. The preview draws into its own cache and warns when the file is stale.

### Memory tables

For struct layouts, packet headers and register bitfields, write a ` ```memory-table ` fence with one JSON object: `label` (the aria-label), `unit` (`"byte"` or `"bit"`), `fields` in order from the lowest offset (`["name (type)", size]`, `["name", size, tone]` for a field in a rainbow wash, `{"pad": n}`, `{"other": "text", "size": n}`), and optional `cols`, `order`, `base`. The build computes the rows, spans and offsets and draws the table as a memory diagram. A raw `<table class="mem-layout">` still works when an `aria-label` or a `<caption>` names it. Each column must fit its number: up to 36 bit columns, and about 24 byte columns.

### Memory layouts

For address spaces, stack frames, pointer chains and region layouts, write a ` ```memory-layout ` fence with one JSON object: `label` (the aria-label) and `regions` from the highest address to the lowest. A region has one of `value`, `word` or `"gap": true`, and optionally `sub`, `start` (its lowest address), `marker`, `id`, `to` (the region its value points into), `fill` (`gap` for padding, or a tone for its light wash, as for the cells of an embedded `list_head`) and `h` (a multiple of the default row height, which follows the column width). The build draws the column, addresses, arrows and `*(value)` labels, and stops on an invalid spec. The two fences in `gdb-kernel-debugging/index.md` are the reference. Check phone width and every theme.

### Canvas

For figures no other form fits, such as a structure beside the memory it points into, bit fields shifted and masked, or bitmap words and the bit an index selects, write a ` ```canvas ` fence with one JSON object: `label`, an optional `title`, and `items` placed on coordinates. Items are `box` (a label, or a `title` with inner `fields`), `row` and `column` (cells sharing borders), `text` (notes), `arrow`, `line` and `dimension`; arrows and lines take points or references such as `"box.right"`, `"row|2.top"` or `{"at": "note.left", "dy": 6}`. The kit's `forms/canvas/prompt.md` lists every field, and its examples draw an arm64 task beside its kernel stack, `pmd_index()` on x86-64 and a bitmap word lookup. An arrow runs straight or diagonal when its ends face each other and bends at most once otherwise, with its text under the horizontal segment. A label that does not fit is a build error that names the size it needs.

### Struct chains

For intrusive lists, where structures link through an embedded member such as `struct list_head` and `container_of` steps back to the structure, write a ` ```struct-chain ` fence with one JSON object: `label`, `fields` (the members from the lowest offset, as in a memory table), `link` (`field`, the embedded member, and `cells`, its pointers with the forward one first), `nodes` (`name` and optional `values`), and optionally `head` and `tones`. A head makes the chain a ring. The build computes the offsets, the `offsetof` dimension on each node and the arrows. A head and three nodes fit the width.

Figures carry no code lines under them; the prose or a code block shows the code.
