# Dev Wiki — Claude Code Schema

## 이 프로젝트는

개발 공부 내용을 축적하는 개인 위키다. Claude Code가 위키를 작성하고 유지관리한다. 사용자는 자료를 제공하고 질문한다.

## 디렉토리 구조

```
dev-wiki/
├── CLAUDE.md              ← 이 파일. 위키 운영 규칙.
├── raw/                   ← 원본 자료 (사용자가 넣음, 수정 금지)
├── wiki/
│   ├── index.md           ← 전체 페이지 목록과 요약
│   ├── sources/           ← 원본 자료 요약
│   ├── concepts/          ← 개념 설명 (closure, event-loop 등)
│   ├── languages/         ← 언어별 정리 (python, typescript 등)
│   ├── tools/             ← 도구/프레임워크 (docker, git, nextjs 등)
│   ├── comparisons/       ← 비교 분석 (REST-vs-GraphQL 등)
│   └── troubleshooting/   ← 삽질 기록과 해결 방법
```

## 페이지 형식

모든 위키 페이지 상단에 YAML frontmatter를 포함한다:

```yaml
---
title: "페이지 제목"
type: concept | language | tool | source | comparison | troubleshooting
tags: [관련태그들]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: []  # 참고한 원본 자료 파일명
---
```

본문 규칙:

- 다른 위키 페이지를 참조할 때 [[페이지명]] 형식 사용 (Obsidian 위키링크)
- 코드 예제를 적극적으로 포함
- 핵심 개념은 "왜"를 먼저 설명하고 "어떻게"를 이어서 설명
- 한국어로 작성

## 워크플로우

### ingest (자료 수집)

사용자가 "raw/에 있는 [파일] 읽고 정리해줘" 또는 "[주제]에 대해 정리해줘"라고 요청하면:

1. 자료를 읽는다 (파일이 있으면 파일을, 없으면 자체 지식 활용)
2. `wiki/sources/`에 요약 페이지를 만든다 (파일 기반인 경우)
3. 관련 concept, language, tool 페이지를 업데이트하거나 새로 만든다
4. 새로 만든 페이지에서 기존 페이지로, 기존 페이지에서 새 페이지로 [[위키링크]]를 추가한다
5. `wiki/index.md`를 갱신한다

### query (질의)

사용자가 질문하면:

1. `wiki/index.md`를 먼저 읽어 관련 페이지를 파악한다
2. 해당 페이지를 읽고 답변한다
3. 답변 중 위키에 남길 가치가 있는 내용은 새 페이지로 저장할지 사용자에게 묻는다

### lint (점검)

사용자가 "lint" 또는 "점검"을 요청하면:

- 고아 페이지 찾기 (아무데서도 링크되지 않은 페이지)
- [[위키링크]]가 걸려 있지만 실제 페이지가 없는 것 찾기
- 교차 참조 누락 확인 (관련 있는데 서로 링크 안 된 페이지)
- 오래된 정보 표시
- 보강할 수 있는 주제 제안
- 결과를 보고하고 수정 여부를 사용자에게 확인받는다

## index.md 형식

카테고리별로 정리하고 한 줄 요약을 붙인다:

```markdown
# Wiki Index

## Concepts
- [[closure]] — 함수가 자신의 렉시컬 스코프를 기억하는 것
- [[event-loop]] — JavaScript 비동기 처리의 핵심 메커니즘

## Languages
- [[python]] — 문법, 관용구, 생태계 정리

## Tools
- [[docker]] — 컨테이너 기반 개발 환경
...
```

## 주의사항

- raw/ 디렉토리의 파일은 절대 수정하지 않는다
- 페이지를 삭제하기 전에 반드시 사용자에게 확인받는다
- 확실하지 않은 내용은 추측하지 말고 모른다고 표시한다
- 기존 페이지를 업데이트할 때 기존 내용을 함부로 지우지 않고 보강한다