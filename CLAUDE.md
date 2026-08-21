# CLAUDE.md

**개발 결과 보고서**를 Jekyll 정적 사이트로 작성·배포하는 리포지토리다.
코드 프로젝트가 아니라 문서 프로젝트이므로, 대부분의 작업은 마크다운 페이지 추가와 레이아웃/스타일 조정이다.

- 사업명(국문): AI 에이전트 기술을 적용한 생활체육 용병 구인 및 실력 검증 플랫폼
- 사업명(영문): AI Agent-powered Amateur Sports Mercenary Recruitment & Skill Verification Platform
- 약칭: 생활체육 AI 에이전트 플랫폼
- 개발자: 정상호, 백성검, 정어진, 박민호 (총 4명)
- 개발기간: 2026-08-20 ~ 2026-10-27 (69일, 약 10주)
- 리포: `jsangho/supersub.jsangho.cloud` (2026-08-21에 `demo.jsangho.cloud`에서 개명. 옛 주소는 GitHub이 리다이렉트해 주지만 새 주소를 쓴다)
- 배포 URL: https://jsangho.github.io/supersub.jsangho.cloud/

---

## 1. 환경

| 항목 | 값 |
|---|---|
| OS | Ubuntu 26.04 LTS (WSL2, 호스트 `DESKTOP-2HAMSV3`) |
| Ruby | 3.3.12 (rbenv, `rbenv global`로 지정됨) |
| Jekyll | 4.4.1 |
| 테마 | minima 2.5.2 (gem) |
| 미리보기 | http://localhost:4000 · http://100.91.86.29:4000 (Tailscale) |

Ruby는 rbenv 기반이므로 `sudo` 없이 gem을 설치한다. 명령은 항상 `bundle exec`를 앞에 붙인다.

```bash
bundle exec jekyll serve --host 0.0.0.0 --port 4000
```

### 미리보기 서비스

상시 미리보기는 systemd **user** unit으로 등록되어 있다.

```bash
systemctl --user status  jekyll-preview.service
systemctl --user restart jekyll-preview.service
```

`_config.yml`은 `jekyll serve`가 자동 리로드하지 **않는다.** 이 파일을 고쳤으면 반드시 서비스를 재시작한다.
`Linger=no` 상태라 로그아웃하면 서비스가 내려간다. 상시 유지가 필요하면 `sudo loginctl enable-linger ho`.

---

## 2. 배포

`main`에 푸시하면 `.github/workflows/jekyll.yml`이 빌드·배포한다. GitHub Pages의 기본 빌더(github-pages gem, Jekyll 3.10)는 **쓰지 않는다** — 이 리포의 Gemfile(Jekyll 4.4.1)과 버전이 어긋나기 때문이다. Pages source는 `build_type: workflow`로 설정되어 있다.

`baseurl`은 `_config.yml`에서 비워 두고, 워크플로의 `configure-pages`가 `--baseurl`로 주입한다. 여기에 값을 하드코딩하면 로컬 미리보기가 깨진다.

### 푸시가 403으로 막히면

환경변수 `GITHUB_TOKEN`(`~/.secrets:3`)에 **읽기 전용** fine-grained PAT가 들어 있고, `~/.gitconfig`의 URL 스코프 헬퍼가 이걸 git에 넘긴다. 에러 메시지가 `Permission to ... denied to jsangho`라 리포 권한 문제처럼 보이지만 아니다.

```bash
GITHUB_TOKEN= git push origin main     # 저장된 gho_ 토큰으로 폴백
GITHUB_TOKEN= gh run list
```

`git -c credential.helper=...`로 덮어쓰는 방식은 URL 스코프 헬퍼가 우선하므로 통하지 않는다.

---

## 3. 문서 작성 규칙

### 3.1 레이아웃 선택

| 레이아웃 | 용도 | 사이트 헤더/푸터 |
|---|---|---|
| `cover` | 표지(`index.md`) 전용 | 없음 (화면 세로 중앙 정렬) |
| `report` | 본문 각 장 | 있음 |
| `page` | 부속 페이지(About 등) | 있음 |

본문 페이지는 `report`를 쓴다. `page`는 minima 기본이라 보고서 타이포그래피가 적용되지 않는다.

### 3.2 본문에 `#` 제목을 쓰지 않는다

`report`·`page` 레이아웃이 front matter의 `title`을 이미 `<h1>`으로 렌더한다. 본문에 `# 제목`을 또 쓰면 같은 제목이 두 번 나온다. 본문 소제목은 `##`부터 시작한다.

```markdown
---
layout: report
title: 사업 개요
---

## 1) 사업 목적      ← ## 부터 시작
```

### 3.3 목차는 중첩 순서 목록으로 쓴다

`1)` `2)` 를 **텍스트로 직접 입력하지 않는다.** 마크다운 리스트로 인식되지 않아 앞 문단에 통째로 붙어버린다(실제로 한 번 깨졌다). 반드시 중첩 `1.` 리스트로 쓰고, 괄호 표기는 CSS가 만든다.

```markdown
1. **사업 개요**
   1. 사업 목적          ← 3칸 들여쓰기
   2. 주요 사업 내용
```

`1)` 형태는 `assets/main.scss`의 `@counter-style paren-decimal`이 렌더한다. `list-style-type`만으로는 괄호 접미사를 표현할 수 없다.

### 3.4 새 장(chapter) 추가 절차

1. 리포 루트에 `NN-slug.md` 생성 (예: `01-overview.md`)
2. front matter에 `layout: report`, `title`, `permalink: /NN-slug/`, `chapter: N`
3. `toc.md`의 해당 항목을 링크로 연결
4. **`header_pages`에는 추가하지 않는다** — 8개 장이 상단 네비를 채우면 못 쓴다. 이동은 목차를 통한다

`chapter`(숫자)는 필수다. `report` 레이아웃이 이 값으로 머리말의 `제 N 장`과 본문 하단 이전/다음 네비를 만든다. 빠뜨리면 그 페이지만 네비 없이 렌더되고, 앞뒤 장의 네비에서도 빠진다.

`toc.md`의 링크는 `baseurl` 때문에 `relative_url`을 거쳐야 한다. 경로를 직접 쓰면 배포본에서 404가 난다.

```markdown
1. **[사업 개요]({{ '/01-overview/' | relative_url }})**
```

### 3.5 front matter

표지(`index.md`)는 내용을 본문이 아니라 front matter에 둔다. 레이아웃이 이 값들을 읽어 렌더하고, 인원수는 배열 길이로 자동 계산된다.

```yaml
members:
  - 정상호
period_start: 2026년 8월 20일(목)
period_note: 69일 · 약 10주
```

팀원 이름·기간이 바뀌면 `index.md`의 이 값만 고친다. 레이아웃은 건드리지 않는다.
영문 사업명은 `index.md`의 `title_en`이며, 표지에서 국문 제목 아래에 부제로 렌더된다.

표지 메타의 `저장소`·`데모` 줄은 `repo_url`·`demo_url`이 만든다. 표기 문자열은 레이아웃이 URL에서 스킴만 떼어 쓰므로 **주소만 고치면 링크와 라벨이 함께 바뀐다.** 값을 비우거나 지우면 그 줄 자체가 렌더되지 않는다.

### 3.5.1 사업명은 세 군데에 나뉘어 있다

국문 사업명이 길어(공백 포함 38자) 자리에 따라 다른 값을 쓴다. 이름이 바뀌면 **네 곳을 함께** 고친다.

| 값 | 위치 | 쓰이는 곳 |
|---|---|---|
| `title` | `_config.yml`, `index.md` | 표지 대제목, `<title>`, og |
| `title_short` | `_config.yml` | 상단 헤더 브랜드, `report` 머리말 |
| `title_en` | `index.md` | 표지 영문 부제 |
| `description` | `_config.yml` | 메타 설명 |

`_includes/header.html`은 minima gem의 동명 파일을 덮어쓴 사본이다. 정식 사업명이 헤더에서 줄바꿈되는 것을 막으려고 `site.title_short`를 쓰는 한 줄만 다르다. gem을 올리면 이 사본도 대조한다.

`_config.yml`의 `tagline`은 지우지 않는다. 표지는 `page.title == site.title`이라 jekyll-seo-tag가 꼬리말로 `description` 전문을 붙이는데, `tagline`이 이걸 `개발 결과 보고서`로 대체한다.

국문 제목·본문의 긴 한글 값에는 `word-break: keep-all`이 걸려 있다(`.cover__title`, `.cover__meta dd`). 어절 중간에서 줄이 끊기지 않게 하는 용도이므로 지우지 않는다.

### 3.6 스타일

CSS는 `assets/main.scss` 한 파일에서만 고친다. 이 파일이 minima gem의 동명 파일을 덮어쓴다. 맨 위 `@import "minima";` 아래에 오버라이드를 추가한다. front matter(`---` 두 줄)를 지우면 Sass 처리가 안 되므로 **삭제 금지**.

디자인 토큰은 파일 상단 변수로 모여 있다(`$ink`, `$muted`, `$accent` 등). 색을 바꿀 때 개별 규칙이 아니라 변수를 고친다.

`@media print` 블록이 있다. 인쇄 시 헤더/푸터와 CTA가 빠지고 표지 뒤에서 페이지가 나뉜다. 레이아웃을 바꾸면 인쇄 결과도 확인한다.

### 3.7 배포 후 확인

GitHub Pages는 `cache-control: max-age=600`이다. 배포 직후 브라우저에 옛 CSS가 남아 스타일이 안 먹은 것처럼 보일 수 있다. 서버 상태를 먼저 확인하고 브라우저를 의심한다.

```bash
curl -sI https://jsangho.github.io/supersub.jsangho.cloud/assets/main.css | head -3
```

---

## 4. 현재 상태

### 완료

- 개발 환경(rbenv·Ruby·Jekyll), 로컬 미리보기, Tailscale 원격 접근, systemd 상시화
- GitHub Actions → Pages 배포 파이프라인
- 표지(`/`), 목차(`/toc/`), `cover`·`report` 레이아웃, 커스텀 스타일
- **본문 8개 장 페이지 생성**(`01-overview.md` ~ `08-appendix.md`) 및 배포 — 2026-08-21
- 목차 → 각 장 링크 연결, 장 하단 이전/다음 네비 (`report` 레이아웃 + `chapter` front matter)

### 남은 작업

- **본문 내용 미작성** — 8개 장의 페이지와 절 구조는 잡혀 있으나, 각 절은 아직 `*작성 예정 — …*` 플레이스홀더다. 실제 작업 기록으로 채워야 한다
- `about.markdown`이 Jekyll 스캐폴드 기본 문구 그대로다 — 교체하거나 `header_pages`에서 제거
- `_config.yml`의 `url`이 비어 있어 `canonical`·`og:url`이 상대 경로로 출력된다 → `url: "https://jsangho.github.io"` 필요
- **커스텀 도메인(`supersub.jsangho.cloud`)이 아직 살아 있지 않다** — DNS가 해석되지 않고 `CNAME` 파일도 없다. 표지의 `데모` 링크(`index.md`의 `demo_url`)는 이미 이 주소를 가리키므로 **현재 눌러도 연결되지 않는다.** 실제로 쓰려면 `CNAME` 파일 + DNS 레코드가 필요하고, 안 쓸 거면 `demo_url`을 Pages 주소로 바꾼다
