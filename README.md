# blog-markdown

블로그 포스트 마크다운 저장소. `blog` 레포의 `content/` 서브모듈로 사용된다.

## 디렉토리 구조

각 폴더 이름이 곧 블로그의 URI 경로로 사용된다.

```
{slug}/
  index.md    # 본문 (Markdown)
  meta.json   # 메타데이터 (title, date, tags, excerpt, readTime 등)
```

예: `rust-async-runtime/` → `/posts/rust-async-runtime/`

## 브랜치

- `main` — 프로덕션 배포용 (Cloudflare Pages)
- `test-content` — AI 생성 테스트 콘텐츠 (로컬 preview 용)
