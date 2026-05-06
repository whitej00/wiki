---
tags: [concept, application-layer]
layer: application
related: [DNS, http, internet-structure]
source_count: 1
last_updated: 2026-04-08
---

# CDN (Content Distribution Network)

지리적으로 분산된 서버 클러스터에 콘텐츠 복사본을 저장하고, 사용자를 가장 적합한 서버로 안내하는 네트워크 인프라. 2020년 기준 인터넷 트래픽의 약 80%가 비디오 스트리밍이며, CDN이 이를 지탱한다.

## 왜 필요한가

단일 데이터 센터 방식의 3가지 문제:
1. 원거리 클라이언트 → 많은 ISP 경유 → 병목 링크 발생
2. 인기 콘텐츠가 같은 링크를 반복 통과 → 대역폭 낭비
3. 단일 장애점 (Single point of failure)

## 서버 배치 철학

### Enter Deep (깊숙이 진입)
- **Akamai** 방식
- access ISP 내부에 소규모 서버 클러스터를 수천 곳에 배치
- 장점: 사용자에 가까움 → 낮은 지연, 높은 처리량
- 단점: 분산된 클러스터 관리가 복잡

### Bring Home (가까이 가져오기)
- **Limelight**, **Level-3** 방식
- IXP(Internet Exchange Point)에 대규모 클러스터를 소수(수십 곳)에 배치
- 장점: 관리 용이, 유지보수 비용 낮음
- 단점: 상대적으로 높은 지연

## CDN 동작 원리

DNS 리다이렉트를 활용한 요청 가로채기:

1. 사용자가 콘텐츠 URL 클릭 (예: `video.netcinema.com/6Y7B23V`)
2. 사용자 호스트 → Local DNS → 콘텐츠 제공자의 Authoritative DNS
3. Authoritative DNS가 CDN 도메인 호스트명을 반환 (예: `a1105.kingcdn.com`)
4. Local DNS → CDN의 DNS 인프라 → 최적 CDN 서버 IP 반환
5. 클라이언트가 해당 CDN 서버에 직접 TCP 연결 + HTTP GET

## 클러스터 선택 전략

- **지리적 근접성(Geographically Closest)**: geo-location DB로 LDNS IP → 지리적 위치 매핑, 가장 가까운 클러스터 선택. 단, 네트워크 경로와 지리적 거리가 다를 수 있음
- **실시간 측정(Real-time Measurements)**: 클러스터에서 LDNS로 주기적 핑/DNS 질의 → 지연 측정 기반 선택

## 사례

### Netflix
- Amazon 클라우드 (사용자 등록, 결제, 카탈로그) + **자체 CDN**
- ISP 내 + IXP에 서버 랙 설치 (200+ IXP 위치)
- **Push 캐싱**: 비성수 시간에 콘텐츠를 CDN 서버에 미리 전송
- DASH 기반 적응적 스트리밍

### YouTube (Google)
- **Google 자체 CDN** (3단계: 메가 데이터센터, IXP 클러스터, access ISP 내 enter-deep 클러스터)
- **Pull 캐싱**: 요청 시 콘텐츠를 가져와 로컬 저장
- DNS 리다이렉트로 클러스터 선택

## DASH (Dynamic Adaptive Streaming over HTTP)

- 비디오를 여러 비트레이트 버전으로 인코딩
- 각 버전의 URL을 **manifest 파일**에 기록
- 클라이언트가 대역폭 측정 후 적절한 품질의 청크를 HTTP GET으로 요청
- 대역폭 변화에 따라 실시간 품질 전환

## 출처

- [[kurose-ch2]] — Kurose 8판 2.6절
