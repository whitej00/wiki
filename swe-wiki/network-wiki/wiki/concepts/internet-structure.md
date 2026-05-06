---
tags: [concept, network]
layer: network
related: [packet-switching, protocol-layers]
source_count: 1
last_updated: 2026-04-08
---

# 인터넷 구조 (Internet Structure)

인터넷은 **네트워크의 네트워크(network of networks)**로, 수십만 개의 ISP를 계층적으로 연결한 복잡한 생태계이다.

## ISP 계층 구조

### Access ISP (접속 ISP)
- 종단 사용자를 인터넷에 연결
- DSL, 케이블, FTTH, WiFi, 셀룰러 등의 접속 기술 제공
- 대학, 기업, 통신사 등이 운영

### Regional ISP (지역 ISP)
- 특정 지역의 여러 access ISP를 연결
- 국가 단위의 더 큰 regional ISP도 존재 (예: 중국의 성(省) 단위 ISP)

### Tier-1 ISP
- 글로벌 백본 네트워크 운영
- 전 세계에 약 12개 존재 (Level 3, AT&T, Sprint, NTT 등)
- 서로 **피어링(peering)** — 정산 없이(settlement-free) 트래픽 교환

## 상호 연결 메커니즘

### 고객-제공자 관계 (Customer-Provider)
- 하위 ISP가 상위 ISP에게 비용을 지불
- 트래픽 양에 비례하여 과금

### 피어링 (Peering)
- 같은 계층의 ISP들이 **직접 연결**하여 트래픽 교환
- 보통 정산 없음 (settlement-free)
- 상위 ISP를 거치지 않으므로 비용 절감

### IXP (Internet Exchange Point)
- 여러 ISP가 모여 피어링하는 물리적 거점
- 전용 스위치가 있는 독립 건물
- 현재 인터넷에 600개 이상 존재

### 멀티호밍 (Multi-homing)
- 하나의 ISP가 2개 이상의 상위 ISP에 동시 연결
- 장애 시에도 인터넷 접속 유지 가능

## 콘텐츠 제공자 네트워크 (Content Provider Network)

Google 등 대형 콘텐츠 제공자는 **자체 사설 네트워크**를 구축:
- 전 세계 데이터 센터를 사설 TCP/IP 네트워크로 연결
- IXP나 하위 ISP와 직접 피어링하여 상위 ISP 우회
- 비용 절감 + 서비스 품질 제어

## 출처

- [[kurose-ch1]] — Kurose 8판 1.3.3절
