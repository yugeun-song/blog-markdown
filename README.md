# blog-markdown

Posts for [vmfault.dev](https://vmfault.dev). The [blog](https://github.com/yugeun-song/blog) engine mounts this repo at `content/`. A push to `main` runs `.github/workflows/notify.yml`, which bumps that submodule and so deploys the site.

## Layout

One directory per post. Its kebab-case ASCII name is the URL slug: `ftrace-usage/` → `/posts/ftrace-usage/`.

- `meta.json`: metadata
- `index.md`: body without frontmatter. The first line is `# Title`.
- `source-snippets.json`: kernel source excerpts, written by the engine's `resolve_kernel.py`
- anything else: assets, referenced by relative path (`./foo.png`) and published next to the page. A PNG may have a lossless `.webp` twin, which browsers get instead.

## meta.json

| Field | Type | Required | Notes |
|---|---|---|---|
| `title` | string | yes | equals the `# Title` line |
| `date` | string | yes | `YYYY-MM-DD`; sort key |
| `tags` | string[] | yes | lowercase ASCII |
| `excerpt` | string | yes | card text and search snippet |
| `series` | string | no | `category/subcategory` such as `linux-kernel/net`; one tier (`bash`) is fine. The URL replaces `/` with `-`. |
| `seriesOrder` | number | no | 1-based topical order (prerequisites first), not chronological |

Tags and series are separate namespaces. Writing rules: [CLAUDE.md](CLAUDE.md).

Single branch `main`; roll back with `git revert`.
