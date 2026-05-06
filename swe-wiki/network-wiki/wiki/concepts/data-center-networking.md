---
tags: [concept, link-layer]
layer: link
related: [Ethernet, link-layer-switching, vlan, ip-addressing]
source_count: 1
last_updated: 2026-04-08
---

# 데이터 센터 네트워킹 (Data Center Networking)

Google, Microsoft, Amazon, Alibaba 등이 운영하는 대규모 데이터 센터의 내부 네트워크. 수만~수십만 호스트를 상호 연결하며, 클라우드 컴퓨팅의 핵심 인프라다.

## 데이터 센터의 역할

1. **콘텐츠 제공** — 웹 페이지, 검색 결과, 이메일, 동영상 스트리밍
2. **대규모 병렬 컴퓨팅** — 검색 인덱싱 등 분산 처리
3. **클라우드 컴퓨팅** — AWS, Azure, Alibaba Cloud 등 IT 인프라 제공

## 물리적 구조

- **블레이드(Blade)** — CPU, 메모리, 디스크를 갖춘 범용 호스트 (피자 박스 형태)
- **랙(Rack)** — 20~40개의 블레이드를 수직 적재
- **TOR 스위치(Top of Rack Switch)** — 랙 최상단에 위치, 랙 내 호스트를 상호 연결하고 상위 스위치와 연결
- 호스트는 TOR 스위치에 40/100 Gbps 이더넷으로 연결

## 로드 밸런싱 (Load Balancing)

- 외부 요청은 먼저 **로드 밸런서**로 향함
- 로드 밸런서가 현재 부하를 기반으로 요청을 적절한 호스트에 분배
- **NAT와 유사한 기능** 수행 — 외부 공개 IP ↔ 내부 호스트 IP 변환
- "Layer-4 스위치"라고도 불림 (목적지 포트 번호 + IP 주소로 결정)

## 계층적 토폴로지

```
          Border Router
               |
          Access Router
           /        \
     Tier-1 SW    Tier-1 SW
      / | \        / | \
  Tier-2 SW      Tier-2 SW
    / | \          / | \
 TOR TOR TOR   TOR TOR TOR
  |   |   |     |   |   |
 Rack Rack Rack Rack Rack Rack
```

- **Border Router** — 데이터 센터를 공용 인터넷에 연결
- **Access Router** — 보더 라우터 아래, 여러 Tier-1 스위치에 연결
- **Tier-1/2 스위치** — 계층적으로 랙을 연결
- **TOR 스위치** — 랙 내 호스트 연결 (3번째 계층)

ARP 브로드캐스트 트래픽 격리를 위해 액세스 라우터 하위 서브넷을 [VLAN](vlan.md)으로 분할.

## 호스트 간 대역폭 문제

같은 랙 내 호스트: TOR 스위치 속도(10 Gbps)로 통신 가능. 그러나 **다른 랙의 호스트 간**에는 상위 스위치 링크를 공유하므로 대역폭이 크게 줄어든다.

예: 40개 랙, 랙 당 10개 호스트, 상위 링크 100 Gbps → 호스트 당 100/40 = 2.5 Gbps

## 해결: 고밀도 상호 연결 (Highly Interconnected Topology)

- 각 TOR을 **여러 Tier-2 스위치**에 연결, 각 Tier-2를 **여러 Tier-1 스위치**에 연결
- 다중 경로로 **용량 증대** + **경로 다양성으로 신뢰성 향상**
- 이러한 다단(multi-stage) 상호 연결 네트워크를 **클로스 네트워크(Clos Network)**라 함
- Facebook: 각 TOR → 4개 Tier-2(서로 다른 "스파인 플레인"), 각 Tier-2 → 4개 Tier-1
- **ECMP(Equal Cost Multi Path, RFC 2992)**로 다중 경로 활용

## 트렌드

### 비용 절감
- 대형 클라우드 업체들이 자체 설계한 **커머디티 스위치** 사용 (off-the-shelf 실리콘)
- 스위치 벤더 구매 대신 인하우스 제작

### SDN 중앙 제어
- Google, Microsoft, Facebook 등이 **SDN 기반 중앙 집중 제어** 채택
- 단순한 커머디티 스위치(데이터 플레인) + 소프트웨어 기반 컨트롤 플레인

### 가상화
- VM(Virtual Machine)이 물리 서버에서 분리 → 다른 랙으로 이동 가능
- ARP를 DNS 스타일 디렉터리로 대체하여 전체 네트워크를 단일 flat L2 네트워크처럼 운용

### 물리적 제약
- 데이터 센터 내 지연: 마이크로초 단위 → TCP/혼잡 제어가 적합하지 않을 수 있음
- RDMA(Remote Direct Memory Access) 기술 등 대안 등장

### 하드웨어 모듈화
- **MDC(Modular Data Center)** — 컨테이너 단위로 데이터 센터 배포
- 장애 시 컨테이너 전체를 교체하는 그레이스풀 디그레이데이션 설계

## 출처

- [kurose-ch6](../sources/kurose-ch6.md) — Kurose 8판 6장 (Section 6.6)
