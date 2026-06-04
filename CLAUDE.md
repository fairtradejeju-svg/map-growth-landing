# CLAUDE.md

Guidance for AI assistants (and humans) working in this repository.

## What this is

**MAP Growth Support Korea** — a static marketing + support website for the NWEA
MAP Growth English assessment program in Korea. The site is published via
**GitHub Pages** to the custom domain in `CNAME` (`www.mapgrowthsupportkorea.com`).

All content is in **Korean (`lang="ko"`)**. The audience is Korean academies
(어학원/학원) and their teachers who administer MAP Growth tests.

There is **no build system, package manager, framework, or test suite**. Every
page is a self-contained, hand-authored HTML file with inline `<style>` and
inline `<script>`. There are no `.js`/`.css` files, no `node_modules`, and no
CI/CD config — what is committed is exactly what is served.

## Repository layout

```
.
├── CNAME                          # GitHub Pages custom domain
├── index.html                     # Main landing page (~1700 lines, the primary file)
├── index_2.html                   # Alternate/older landing variant (not linked from index.html)
├── teacher-guide.html             # Standalone 교사 가이드 (teacher guide) page
├── MAP_Growth_사용매뉴얼.html      # 사용 매뉴얼 hub — opens the guides/ pages in a modal iframe
├── guides/                        # 9 self-contained how-to guide pages (loaded by the manual)
│   ├── 01. 학생 등록 (로스터링) 가이드.html
│   ├── 02. 시험 세션 만들기 가이드.html
│   ├── 03. 학생 로그인 안내.html
│   ├── 04. 시험 중 관리 가이드.html
│   ├── 04-2. 시험 종료 가이드.html
│   ├── 05. 트러블슈팅 & 문제 해결.html
│   ├── 06. 리포트 결과 확인 가이드.html
│   ├── 07. 로스터링 오류 해결.html
│   └── 08. 빠른 접속 참고자료.html
├── wb/
│   └── 1-06/index.html            # Standalone interactive workbook (SIR Book 1-06 "So Many Sounds")
├── images/
│   └── map-logo.jpg               # Logo asset (note: index.html also embeds a base64 logo inline)
├── *.pdf                          # Reference/sample PDFs (report sample, testing tips, password reset, setup)
└── files.zip                      # Archived snapshot of index.html + teacher-guide.html (not served)
```

### How the pages relate

- **`index.html`** is the entry point. Its top nav links to `#intro`, `#services`,
  `#analyzer`, `#contact` (in-page anchors), plus `teacher-guide.html` and
  `MAP_Growth_사용매뉴얼.html`.
- **`MAP_Growth_사용매뉴얼.html`** is a hub: category cards call
  `openGuide(file, title, catIdx)`, which loads `guides/<file>` into a modal
  `<iframe>` (`basePath = 'guides/'`). The guide pages in `guides/` are designed
  to render inside that iframe.
- **`teacher-guide.html`** links back to `index.html` and `index.html#intro`.
- **`wb/1-06/index.html`** is an independent interactive student workbook and is
  not linked from the main nav.
- **`index_2.html`** and **`files.zip`** are not referenced by any live page —
  treat them as legacy/backup unless told otherwise. Do not assume edits to
  `index.html` should be mirrored there.

## Tech conventions

Follow the existing patterns in whatever file you are editing — they are very
consistent across pages:

- **Single-file pages.** HTML, CSS (`<style>` in `<head>`), and JS (`<script>`
  before `</body>`) all live in one file. Do not extract them into separate
  files or introduce a bundler.
- **Theming via CSS custom properties.** The `<html>` element carries
  `class="theme-light"` (or `theme-dark`). Color tokens are defined under
  `html.theme-light { ... }` / `html.theme-dark { ... }` as variables like
  `--bg`, `--surface`, `--text`, `--accent` (`#0F6E56`, the brand green),
  `--accent-secondary`, `--positive`, `--negative`, `--warning`. Always style
  with `var(--token)` rather than hardcoded colors so both themes work.
- **Fonts.** Google Fonts `Noto Sans KR` (primary, Korean) + `Inter`, loaded via
  `<link>` from `fonts.googleapis.com`.
- **Typography scale** uses `clamp()` for fluid sizing (e.g. `h1 { font-size: clamp(2.5rem, 6vw, 4rem) }`).
- **Layout** uses `.container { max-width: 1120px; margin: 0 auto; padding: 0 24px; }`.
- **Vanilla JS only**, written in ES5/ES6 style with `var`/`const`, plain
  `document.getElementById` / `querySelector`, and inline `onclick=`/event
  listeners. No frameworks, no jQuery, no modules.
- **Single external script dependency:** `html-to-image` (CDN, jsDelivr) — used
  by the `downloadImage()` feature to export rendered report mockups as images.
- **SVG icons are inline** (hand-written `<svg>`, typically `stroke="currentColor"`,
  `stroke-width="2"`, 24×24 viewBox). Reuse this style for new icons.
- **`data-reveal`** attributes mark sections animated in on scroll via an
  `IntersectionObserver` in `index.html`.

## Key interactive features in `index.html`

The inline `<script>` (roughly lines 1400–1730) contains the page logic. Notable pieces:

- **Theme toggle** — `applyTheme()` / `cycleTheme()`, persisted in `localStorage`.
- **Mobile nav** — `toggleMenu()` toggles `.nav-links.open`.
- **Slideshow** (`#intro` section) — `go()`, `play()`, `stop()`, `togglePlay()`
  with dots/counter controls (`#bp`, `#pp`, `#dots`, `#bn`, `#cnt`).
- **Counters** — `animateCounters()` / `tick()` count-up animation.
- **Report image export** — `downloadImage()` uses the `html-to-image` lib.
- **Interactive Korea map of partner centers** (`#centers` section). The source
  of truth is the **`CENTER_DATA`** array (each region: `id`, `label`, `sub`,
  `count`, and a `centers[]` list of `{ name, district, addr, phone }`).
  `renderCenterList()`, `selectCenterRegion()`, and `closeCenterDetail()` render
  the searchable list (`#centerSearch`) and detail panel from that array, with
  `_hl()` highlighting search matches. **When asked to add/update a partner
  academy (학원) — name, address, phone — edit the `CENTER_DATA` array and keep
  the region `count` in sync with its `centers[]` length.**

## Editing guidance

- **Match the file you are in.** Indentation, inline-style density, Korean
  microcopy tone, and icon style should mirror the surrounding code. These pages
  were authored by hand and visual consistency matters.
- **Keep content in Korean** unless explicitly asked otherwise.
- **Preserve both light and dark themes** — verify any new color uses a `var(--…)`
  token that exists in both blocks.
- **Filenames in `guides/` contain Korean text, spaces, and numbering** (`01. …`).
  If you rename or add a guide, also update the matching `openGuide('<file>', …)`
  call in `MAP_Growth_사용매뉴얼.html` (note that `&` in a filename is written as
  `%26` in those `onclick` strings, e.g. `'05. 트러블슈팅 %26 문제 해결.html'`).
- **`index_2.html` / `files.zip` are not live.** Don't edit them to "keep things
  in sync" unless the task explicitly says so.

## Running / previewing locally

No build step. Serve the directory with any static file server and open in a
browser, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

Opening `index.html` directly via `file://` mostly works, but the
`MAP_Growth_사용매뉴얼.html` guide modal loads `guides/*.html` into an iframe and
is more reliable over `http://`. The `html-to-image` export and Google Fonts
also require network access.

## Deployment

Pushing to the default branch publishes via **GitHub Pages** to the domain in
`CNAME`. There is no build pipeline — committed files are served as-is. After
making changes, the live site reflects exactly what was pushed.

## Git workflow

- Commit messages in this repo are concise and follow a loose
  `type: 설명` convention (`feat:`, `fix:`, `update:`), with the description
  usually written in **Korean** (e.g. `feat: SIR Book 1-06 So Many Sounds 워크북 추가`,
  `fix: 모바일 햄버거 메뉴 버그 수정`). Match that style.
- Do unsolicited work on a feature branch, not the default branch; open a PR
  rather than pushing straight to `main`.
