# Computer Network Wiki — Schema

이 위키는 컴퓨터 네트워크 지식을 체계적으로 축적하기 위한 LLM 관리형 위키입니다.

## 디렉토리 구조

```
raw/                # 원본 소스 (수정 금지)
  assets/           # 이미지, 다이어그램
wiki/               # LLM이 관리하는 마크다운 파일
  index.md          # 위키 전체 목록
  overview.md       # 네트워크 전체 개관
  layers/           # OSI/TCP-IP 계층별 페이지
  protocols/        # 프로토콜별 페이지
  concepts/         # 핵심 개념 페이지
  comparisons/      # 비교/분석 페이지
  sources/          # 소스 요약 페이지
```

## 페이지 종류

### 1. 프로토콜 페이지 (`protocols/`)
파일명: `protocols/tcp.md`, `protocols/bgp.md` 등

프론트매터:
```yaml
---
tags: [protocol, transport-layer]
layer: transport
rfc: 793
related: [UDP, IP, congestion-control]
source_count: 0
last_updated: 2026-04-08
---
```

내용 구성:
- 한 줄 요약
- 목적과 동작 원리
- 헤더 구조 / 포맷 (있을 경우)
- 주요 메커니즘 (e.g. TCP의 3-way handshake, congestion control)
- 관련 프로토콜과의 관계
- 출처 (어떤 소스에서 왔는지)

### 2. 개념 페이지 (`concepts/`)
파일명: `concepts/congestion-control.md`, `concepts/subnetting.md` 등

프론트매터:
```yaml
---
tags: [concept, transport-layer]
layer: transport
related: [TCP, AIMD, slow-start]
source_count: 0
last_updated: 2026-04-08
---
```

내용 구성:
- 정의
- 왜 필요한지 (문제 배경)
- 핵심 메커니즘 / 알고리즘
- 관련 프로토콜 / 개념 링크
- 출처

### 3. 계층 페이지 (`layers/`)
파일명: `layers/transport.md`, `layers/network.md` 등

각 계층의 역할, 포함된 프로토콜/개념 목록, 계층 간 관계를 정리.

### 4. 비교 페이지 (`comparisons/`)
파일명: `comparisons/tcp-vs-udp.md` 등

Query에서 나온 분석이나 비교를 영구 보존할 때 사용.

### 5. 소스 요약 (`sources/`)
파일명: `sources/kurose-ch3.md`, `sources/rfc793.md` 등

원본 소스 1개당 요약 1개. 핵심 내용, 다룬 토픽 목록, 위키에 반영된 페이지 목록 포함.

## 컨벤션

- 파일명: 소문자 kebab-case (`congestion-control.md`)
- 내부 링크: Obsidian 위키링크 사용 (`[TCP](wiki/protocols/tcp.md)`, `[congestion-control](wiki/concepts/congestion-control.md)`)
- 계층 참조: OSI 7계층과 TCP/IP 4계층 모델 병기. `layer` 프론트매터에는 TCP/IP 기준 사용 (application, transport, network, link)
- 약어: 페이지 제목은 약어 사용 가능 (TCP, BGP 등). 본문 첫 등장 시 풀네임 병기.
- 언어: 한국어로 작성. 기술 용어는 영문 병기 (e.g. 혼잡 제어(Congestion Control)).

## 워크플로우

### Ingest (소스 인제스트)

1. `raw/`에 있는 소스를 읽는다.
2. 사용자와 핵심 내용을 논의한다.
3. `sources/`에 소스 요약 페이지를 생성한다.
4. 관련 프로토콜/개념/계층 페이지를 생성하거나 업데이트한다.
   - 기존 페이지와 **충돌**하는 내용이 있으면 명시적으로 표기한다: `> [!warning] 충돌: ...`
   - 새로운 내용이 기존 설명을 **보강**하면 출처를 추가하며 병합한다.
5. `index.md`를 업데이트한다.

### Query (질문 응답)

1. `index.md`를 읽어 관련 페이지를 찾는다.
2. 해당 페이지들을 읽고 종합하여 답변한다.
3. 답변에는 위키 페이지 출처를 명시한다.
4. 가치 있는 답변(비교, 분석 등)은 사용자 동의 하에 `comparisons/`나 `concepts/`에 저장한다.

### Lint (위키 점검)

주기적으로 다음을 점검한다:
- 페이지 간 모순
- 인바운드 링크 없는 고아 페이지
- 언급만 되고 페이지가 없는 개념 ([missing-page](#missing-page))
- 빠진 cross-reference
- 프론트매터 누락/불일치
- source_count가 0인 페이지 (출처 없는 주장)

## index.md 형식

```markdown
# 위키 인덱스

## 계층 (Layers)
- [transport](wiki/layers/transport.md) — 전송 계층 개관

## 프로토콜 (Protocols)
- [TCP](wiki/protocols/tcp.md) — 신뢰성 있는 전송 프로토콜
- [UDP](wiki/protocols/udp.md) — 비연결형 전송 프로토콜

## 개념 (Concepts)
- [congestion-control](wiki/concepts/congestion-control.md) — 혼잡 제어 메커니즘

## 비교 (Comparisons)
- [tcp-vs-udp](#tcp-vs-udp) — TCP와 UDP 비교

## 소스 (Sources)
- [kurose-ch3](wiki/sources/kurose-ch3.md) — Kurose 교재 3장 요약
```
