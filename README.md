# blog-markdown

[devlog](https://github.com/yugeun-song/blog) 블로그의 콘텐츠 저장소. 메인 블로그 프로젝트의 `content/` 경로에 git 서브모듈로 마운트된다. `main`에 push하면 GitHub Actions가 blog 저장소의 서브모듈 포인터를 자동 갱신해 Cloudflare Pages 빌드를 트리거한다.

## 디렉토리 구조

각 포스트는 slug 디렉토리 하나로 구성된다:

- `{slug}/meta.json` — 메타데이터
- `{slug}/index.md` — 마크다운 본문(frontmatter 없음, `# Title`로 시작)

디렉토리 이름이 그대로 URL 경로가 된다: `rust-async-runtime/` → `https://blog-213.pages.dev/posts/rust-async-runtime/`

슬러그는 kebab-case ASCII만 사용한다.

## `meta.json` 스키마

```json
{
  "title": "CFS 스케줄러 깊이 읽기",
  "date": "2026-03-01",
  "tags": ["linux", "kernel", "scheduler"],
  "excerpt": "한 줄 요약",
  "series": "linux-kernel/process",
  "seriesOrder": 3
}
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `title` | string | yes | 페이지 제목, 카드 타이틀, `<title>` |
| `date` | string | yes | `YYYY-MM-DD`. 정렬 키 |
| `tags` | string[] | yes | 필터링과 태그 클라우드. 소문자 ASCII 권장 |
| `excerpt` | string | yes | 카드 미리보기, 검색 결과 스니펫 |
| `series` | string | no | `category/subcategory` 형식. 표시명은 그대로, URL 슬러그는 `/`를 `-`로 치환 |
| `seriesOrder` | number | no | topical(선수 학습 먼저) 순서. 1-based, 시간순 아님 |

## 시리즈 명명 규칙

소문자 + 하이픈 + `/` 계층 구분자. 빌드 시 URL 슬러그로는 `/`가 `-`로 바뀐다.

- `linux-kernel/net`
- `linux-kernel/process`
- `linux-system-programming/io`
- `math/information-theory`
- `math/optimization`
- `bash` (계층 없는 단일 시리즈)

태그와 시리즈는 서로 독립된 네임스페이스를 가진다.

## `index.md` 작성 규칙

- frontmatter 없음(`meta.json`이 메타데이터를 소유한다)
- 첫 줄은 `# Title`이며 `meta.json.title`과 일치해야 한다
- `h2` / `h3`은 빌드 타임에 자동으로 TOC에 등록된다(`rehype-slug`가 id 부여)
- 코드 블록에는 항상 언어 태그를 붙인다(`c`, `python`, `bash`, `text` 등). 설정/로그/의사코드에는 `text`를 쓴다
- GFM 지원: 테이블, strikethrough, task list

### 수식 (KaTeX)

인라인 `$...$`, 디스플레이 `$$...$$`. 빌드 타임 SSR이라 클라이언트 JS 없이 바로 렌더링된다. 마크다운 테이블 셀 안에서는 `|` 대신 `\lvert` / `\rvert`를 사용해야 파이프 파서와 충돌하지 않는다.

### 다이어그램 (Mermaid)

```` ```mermaid ```` 코드 펜스로 작성한다. 클라이언트 측에서 렌더링되며 4개 테마와 연동된다. 노드 라벨은 함수명/키워드 한 줄로 짧게 두고, 설명은 본문에 배치한다. 노드 클래스: `:::accent`, `:::info`, `:::warn`, `:::danger`, `:::muted`.

### 메모리 레이아웃 테이블

원시 HTML `<table class="mem-layout">`을 마크다운 안에 직접 쓴다. 빌드 타임에 `<div class="table-scroll">`로 감싸진 뒤, 클라이언트 JS가 반응형 인라인 SVG로 변환한다.

- 셀 클래스: `field`(데이터), `pad`(패딩 바이트), `offset`(오프셋 열), 헤더는 `<th>`
- `colspan`으로 멀티바이트/멀티비트 필드를 표현한다
- 클릭하면 확장 모달에서 확대/축소/팬 가능

## 브랜치 정책

`main` 단일 브랜치. 기능 브랜치를 만들지 않고 `main`에 직접 커밋·푸시한다. 롤백은 `git revert` 또는 `git restore`로 처리한다.

## 자동 배포 워크플로우

`.github/workflows/notify.yml`이 `main` 브랜치 push 이벤트에 반응해 `yugeun-song/blog` 저장소를 체크아웃하고 `git submodule update --remote content`를 실행한다. 변경사항이 있으면 `Update content submodule` 커밋을 푸시하고, 이 커밋이 Cloudflare Pages의 자동 빌드를 트리거한다.
