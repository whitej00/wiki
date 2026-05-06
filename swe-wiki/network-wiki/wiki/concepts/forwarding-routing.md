---
tags: [concept, network-layer]
layer: network
related: [router-architecture, generalized-forwarding, ip-addressing, network]
source_count: 1
last_updated: 2026-04-08
---

# 포워딩과 라우팅 (Forwarding & Routing)

네트워크 계층의 두 가지 핵심 기능. 흔히 혼용되지만 범위와 시간 척도가 근본적으로 다르다.

## 포워딩 (Forwarding) — Data Plane

패킷이 라우터의 입력 링크에 도착했을 때, 해당 패킷을 적절한 **출력 링크**로 전달하는 **라우터 로컬(per-router)** 동작이다.

- **시간 척도**: 나노초
- **구현**: 하드웨어 (라인 카드)
- **핵심 데이터 구조**: **포워딩 테이블(Forwarding Table)**
- 도착 패킷의 헤더 필드 값을 포워딩 테이블에서 조회하여 출력 포트 결정

### 목적지 기반 포워딩 (Destination-based)

목적지 IP 주소만으로 출력 포트를 결정한다. [[ip-addressing|최장 프리픽스 매칭(Longest Prefix Matching)]]을 사용.

### 일반화된 포워딩 (Generalized Forwarding)

→ 상세: [[generalized-forwarding]]

목적지 주소뿐 아니라 다양한 헤더 필드(출발지 IP, 포트, 프로토콜 등)를 기반으로 포워딩/드롭/수정 등의 동작을 수행. SDN과 OpenFlow가 이 방식을 구현.

## 라우팅 (Routing) — Control Plane

패킷이 송신 호스트에서 수신 호스트까지 취할 **종단간(end-to-end) 경로**를 결정하는 **네트워크 전체(network-wide)** 프로세스이다.

- **시간 척도**: 초
- **구현**: 소프트웨어 (라우팅 프로세서)
- **핵심 산출물**: 포워딩 테이블의 내용을 결정
- 라우팅 알고리즘(Dijkstra, Bellman-Ford)과 라우팅 프로토콜(OSPF, BGP)이 이를 수행

## Control Plane의 두 접근

### 1. 전통적 접근 (Per-router Control)

각 라우터에서 라우팅 알고리즘이 실행되고, 이웃 라우터와 라우팅 메시지를 교환하여 자신의 포워딩 테이블을 계산한다. OSPF, BGP가 이 방식.

### 2. SDN 접근 (Logically Centralized Control)

원격의 **SDN 컨트롤러**가 포워딩 테이블을 계산하고 각 라우터에 배포한다. 라우터는 포워딩만 수행. 컨트롤러와 라우터는 잘 정의된 프로토콜(예: OpenFlow)로 통신.

장점: 네트워크 전체 관점에서 최적화 가능, 소프트웨어 업데이트 용이
사례: Google B4, SWAN(Microsoft), ISP 배포(COMCAST, Deutsche Telecom)

## 비유

여행에 비유하면:
- **포워딩** = 교차로에서 어느 방향으로 갈지 결정하는 것
- **라우팅** = 출발 전 지도를 보고 전체 경로를 계획하는 것

## 출처
- [[kurose-ch4]]
