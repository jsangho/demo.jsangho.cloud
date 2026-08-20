# CLAUDE.md

Title Demo 사업의 **개발 결과 보고서**를 Jekyll 정적 사이트로 작성·배포하는 리포지토리다.
코드 프로젝트가 아니라 문서 프로젝트이므로, 대부분의 작업은 마크다운 페이지 추가와 레이아웃/스타일 조정이다.

- 사업명: Title Demo
- 개발자: 정상호, 백성검, 정어진, 박민호 (총 4명)
- 개발기간: 2026-08-20 ~ 2026-10-27 (69일, 약 10주)
- 배포 URL: https://jsangho.github.io/demo.jsangho.cloud/

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
2. front matter에 `layout: report`, `title`, `permalink: /NN-slug/`
3. `toc.md`의 해당 항목을 링크로 연결
4. **`header_pages`에는 추가하지 않는다** — 8개 장이 상단 네비를 채우면 못 쓴다. 이동은 목차를 통한다

### 3.5 front matter

표지(`index.md`)는 내용을 본문이 아니라 front matter에 둔다. 레이아웃이 이 값들을 읽어 렌더하고, 인원수는 배열 길이로 자동 계산된다.

```yaml
members:
  - 정상호
period_start: 2026년 8월 20일(목)
period_note: 69일 · 약 10주
```

팀원 이름·기간이 바뀌면 `index.md`의 이 값만 고친다. 레이아웃은 건드리지 않는다.

### 3.6 스타일

CSS는 `assets/main.scss` 한 파일에서만 고친다. 이 파일이 minima gem의 동명 파일을 덮어쓴다. 맨 위 `@import "minima";` 아래에 오버라이드를 추가한다. front matter(`---` 두 줄)를 지우면 Sass 처리가 안 되므로 **삭제 금지**.

디자인 토큰은 파일 상단 변수로 모여 있다(`$ink`, `$muted`, `$accent` 등). 색을 바꿀 때 개별 규칙이 아니라 변수를 고친다.

`@media print` 블록이 있다. 인쇄 시 헤더/푸터와 CTA가 빠지고 표지 뒤에서 페이지가 나뉜다. 레이아웃을 바꾸면 인쇄 결과도 확인한다.

### 3.7 배포 후 확인

GitHub Pages는 `cache-control: max-age=600`이다. 배포 직후 브라우저에 옛 CSS가 남아 스타일이 안 먹은 것처럼 보일 수 있다. 서버 상태를 먼저 확인하고 브라우저를 의심한다.

```bash
curl -sI https://jsangho.github.io/demo.jsangho.cloud/assets/main.css | head -3
```

---

## 4. 현재 상태

### 완료

- 개발 환경(rbenv·Ruby·Jekyll), 로컬 미리보기, Tailscale 원격 접근, systemd 상시화
- GitHub Actions → Pages 배포 파이프라인
- 표지(`/`), 목차(`/toc/`), `cover`·`report` 레이아웃, 커스텀 스타일

### 남은 작업

- **본문 8개 장 미작성** — 현재 목차만 있고 실제 내용 페이지가 없다
- `toc.md`의 항목이 아직 링크가 아니다 (장 페이지 생성 후 연결)
- `about.markdown`이 Jekyll 스캐폴드 기본 문구 그대로다 — 교체하거나 `header_pages`에서 제거
- `_config.yml`의 `url`이 비어 있어 `canonical`·`og:url`이 상대 경로로 출력된다 → `url: "https://jsangho.github.io"` 필요
- 커스텀 도메인(`demo.jsangho.cloud`) 사용 여부 미정 — 쓰려면 `CNAME` 파일과 DNS 레코드 필요
