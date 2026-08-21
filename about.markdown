---
layout: report
title: 문서 정보
permalink: /about/
---

{%- assign cover = site.pages | where: "permalink", "/" | first -%}

이 사이트는 **{{ site.title }}** 사업의 개발 결과 보고서다. 사업의 배경과 요구사항, 설계 판단, 시험 계획과 결과를 한 문서에 모아 둔 것으로, 별도의 산출물이 아니라 이 사업의 공식 보고서다.

## 1) 사업 정보

| 항목 | 내용 |
|---|---|
| 사업명 | {{ site.title }} |
| 영문명 | {{ cover.title_en }} |
| 팀명 | {{ cover.team }} · {{ cover.team_ko }} |
| 개발자 | {{ cover.members | join: " · " }} (총 {{ cover.members | size }}명) |
| 개발기간 | {{ cover.period_start }} ~ {{ cover.period_end }} ({{ cover.period_note }}) |
| 저장소 | [{{ cover.repo_url | remove: "https://" }}]({{ cover.repo_url }}) |
| 데모 | [{{ cover.demo_url | remove: "https://" }}]({{ cover.demo_url }}) |

## 2) 문서 구성

본문은 9개 장으로 이루어진다. 전체 목차는 [목차]({{ '/toc/' | relative_url }})에 있다.

| 장 | 다루는 내용 |
|---|---|
| [1. 사업 개요]({{ '/01-overview/' | relative_url }}) | 추진 배경, 목적, 개발 범위와 제외 범위, 기대 효과 |
| [2. 요구사항 분석]({{ '/02-requirements/' | relative_url }}) | 문제 정의, 사용자·기능·비기능 요구사항 |
| [3. 시스템 설계]({{ '/03-architecture/' | relative_url }}) | 시스템 구성, 기술 스택, 데이터 모델, 화면 |
| [4. AI 에이전트 설계]({{ '/04-agent/' | relative_url }}) | 영상 분석·매칭·검증 에이전트, 도구·프롬프트, 모델 선정 |
| [5. 용병 구인 기능 개발]({{ '/05-recruitment/' | relative_url }}) | 공고 등록, 자동 매칭, 참여 확정과 노쇼 관리 |
| [6. 실력 검증 기능 개발]({{ '/06-verification/' | relative_url }}) | 스탯 체계, RAG 기반 근거 검증, 신뢰도 산출 |
| [7. 시험 및 검증 결과]({{ '/07-testing/' | relative_url }}) | 시험 항목과 판정 기준, 측정 결과, 이슈 |
| [8. 개발 일정 및 추진 체계]({{ '/08-schedule/' | relative_url }}) | 스프린트 계획, 칸반 보드, 역할 분담, 위험 관리 |
| [9. 결론 및 향후 과제]({{ '/09-conclusion/' | relative_url }}) | 결과 요약, 한계, 향후 계획, 참고 자료 |

## 3) 문서를 읽을 때

### 아직 완성된 문서가 아니다

사업이 진행 중이므로 장마다 완성도가 다르다.

| 상태 | 해당 장 |
|---|---|
| 초안 작성 | 1~6장, 8장 |
| 골격만 (결과 대기) | 7장, 9장 |

본문은 **구현이 시작되기 전 단계의 설계 서술**이다. 화면 캡처, API 경로, 성능 실측치는 아직 들어 있지 않다.

### 〔확인 필요〕 표기

팀에서 결정되지 않았거나 실측이 필요한 값은 본문에 `〔확인 필요: …〕`로 표시해 두었다. 잠정값을 채워 넣고 결정된 것처럼 보이게 하지 않기 위해서다. **이 표기가 붙은 값은 확정된 값이 아니다.**

### 숫자의 출처

지표와 목표치는 [제 7 장]({{ '/07-testing/' | relative_url }})의 측정 결과를 원본으로 삼는다. 다른 장에 같은 숫자가 나오면 7장에서 옮겨 온 값이며, 각 장에서 따로 계산하지 않는다.

## 4) 이 사이트에 대하여

정적 사이트 생성기 [Jekyll](https://jekyllrb.com/)로 만들고 GitHub Pages로 배포한다. `main` 브랜치에 반영되면 GitHub Actions가 빌드해 자동으로 게시한다.

문서의 원본(마크다운)과 사이트 소스는 모두 [저장소]({{ cover.repo_url }})에 공개되어 있으며, 각 장의 변경 이력은 커밋 기록에서 확인할 수 있다.

인쇄를 염두에 두고 스타일을 맞춰 두었다. 브라우저에서 인쇄하면 상단 메뉴와 이동 링크가 빠지고 본문만 출력된다.

[&larr; 표지]({{ '/' | relative_url }})
{:.toc-back}
