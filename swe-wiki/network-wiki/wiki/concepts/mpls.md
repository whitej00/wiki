---
tags: [concept, link-layer, network-layer]
layer: link
rfc: [3031, 3032]
related: [forwarding-routing, ip-addressing, generalized-forwarding]
source_count: 1
last_updated: 2026-04-08
---

# MPLS (Multiprotocol Label Switching)

**고정 길이 라벨을 기반으로 패킷을 포워딩**하는 기술. IP 주소 대신 라벨을 사용하여 포워딩 속도를 높이고, IP 라우팅만으로는 불가능한 **트래픽 엔지니어링**을 가능하게 한다. RFC 3031, RFC 3032에 정의.

## 배경과 동기

1990년대 중반, IP 라우터의 포워딩 속도를 개선하기 위해 가상 회선 개념의 고정 길이 라벨을 도입. 목적지 IP 기반 포워딩 인프라를 대체하는 것이 아니라, 선택적으로 라벨을 부여하여 **보강(augment)**하는 접근.

## MPLS 헤더

이더넷/PPP 헤더와 IP 헤더 **사이**에 위치하는 소형 헤더:

```
| Ethernet Header | MPLS Header | IP Header | Payload |
                   | Label | Exp | S | TTL |
```

| 필드 | 크기 | 설명 |
|------|------|------|
| Label | 20비트 | 포워딩에 사용되는 라벨 값 |
| Exp | 3비트 | 실험용 (QoS 등) |
| S | 1비트 | 스택 바닥 표시 (MPLS 헤더 스택 지원) |
| TTL | 8비트 | Time-to-live |

## 라벨 스위칭 동작

MPLS 지원 라우터는 **라벨 스위치 라우터(Label-Switched Router)**라 불린다:

1. MPLS 프레임 수신 시 **라벨 값만 확인** (IP 주소 추출이나 최장 프리픽스 매칭 불필요)
2. 포워딩 테이블에서 라벨 → (출력 라벨, 목적지, 출력 인터페이스) 매핑 조회
3. 라벨을 교체(swap)하고 해당 인터페이스로 포워딩

> MPLS 프레임은 MPLS 지원 라우터 사이에서만 전송 가능. 비-MPLS 라우터가 수신하면 혼란이 발생.

## 트래픽 엔지니어링

MPLS의 핵심 가치는 속도 향상보다 **트래픽 관리 능력**에 있다:

- 같은 목적지로 가는 **다중 MPLS 경로** 설정 가능
  - 예: 라우터 R4에서 목적지 A로 가는 경로가 2개 (라벨 10: 인터페이스 0, 라벨 8: 인터페이스 1)
- IP 라우팅(최소 비용 경로만 사용)으로는 불가능한 **부하 분산, 경로 분리** 가능
- 네트워크 운영자가 트래픽을 특정 경로로 강제 우회 가능 (정책/성능 기반)

## 기타 활용

- **빠른 장애 복구**: 링크 장애 시 사전 계산된 대체 MPLS 경로로 즉시 전환
- **VPN(Virtual Private Network)**: ISP가 MPLS로 고객의 네트워크를 연결하여 자원과 주소 공간을 격리

## MPLS vs SDN

MPLS의 트래픽 엔지니어링 기능은 SDN과 [[generalized-forwarding|일반화된 포워딩]]으로도 구현 가능. SDN 같은 신기술이 MPLS를 대체할지, 공존할지는 아직 미정.

## 출처

- [[kurose-ch6]] — Kurose 8판 6장 (Section 6.5)
