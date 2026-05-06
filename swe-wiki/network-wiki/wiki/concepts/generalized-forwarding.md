---
tags: [concept, network-layer]
layer: network
related: [forwarding-routing, router-architecture, middleboxes]
source_count: 1
last_updated: 2026-04-08
---

# 일반화된 포워딩과 SDN (Generalized Forwarding & SDN)

목적지 IP 주소만으로 포워딩하는 전통적 방식을 넘어, 패킷 헤더의 **다양한 필드**를 기반으로 포워딩/드롭/수정 등의 동작을 수행하는 **match-plus-action** 패러다임이다. **SDN(Software-Defined Networking)**이 이를 구현한다.

## Match-Plus-Action

각 라우터(패킷 스위치)는 원격 컨트롤러가 계산하여 배포한 **플로우 테이블(Flow Table)**을 유지한다. 플로우 테이블의 각 엔트리는:

1. **매칭 필드 집합**: 도착 패킷의 헤더 값과 비교
2. **카운터 집합**: 매칭된 패킷 수, 마지막 매칭 시간 등
3. **액션 집합**: 매칭 시 수행할 동작

목적지 기반 포워딩은 match-plus-action의 **특수한 경우**이다.

## OpenFlow

SDN의 핵심 표준. 플로우 테이블 추상화를 정의한다.

### 매칭 가능 필드 (OpenFlow 1.0)

링크·네트워크·전송 **3개 계층**의 12개 필드를 매칭할 수 있다:

| 계층 | 필드 |
|------|------|
| 링크 (L2) | Ingress Port, Src MAC, Dst MAC, Eth Type, VLAN ID, VLAN Pri |
| 네트워크 (L3) | IP Src, IP Dst, IP Proto, IP TOS |
| 전송 (L4) | TCP/UDP Src Port, TCP/UDP Dst Port |

- 와일드카드 사용 가능 (예: `128.119.*.*`)
- 여러 엔트리 매칭 시 **우선순위(priority)**가 높은 엔트리 적용
- 매칭되는 엔트리가 없으면 패킷을 드롭하거나 컨트롤러로 전송

### 주요 액션

- **Forwarding**: 특정 출력 포트로 전달, 브로드캐스트, 멀티캐스트, 컨트롤러로 캡슐화 전송
- **Dropping**: 매칭 시 아무 액션 없으면 패킷 폐기
- **Modify-field**: 10개 헤더 필드(L2/3/4)를 재작성 가능

## 활용 예시

### 1. 단순 포워딩
특정 서브넷의 패킷을 지정된 출력 포트로 전달.
```
Match: IP Src=10.3.*.*, IP Dst=10.2.*.*  →  Action: Forward(4)
```

### 2. 로드 밸런싱
출발지 호스트에 따라 서로 다른 경로로 트래픽 분산. 목적지 기반 포워딩으로는 불가능.
```
Match: Ingress=3, IP Dst=10.1.*.*  →  Action: Forward(2)
Match: Ingress=4, IP Dst=10.1.*.*  →  Action: Forward(1)
```

### 3. 방화벽
특정 출발지의 트래픽만 허용.
```
Match: IP Src=10.3.*.*, IP Dst=10.2.0.3  →  Action: Forward(3)
(다른 출발지 → 엔트리 없음 → Drop)
```

## SDN의 의의

- 포워딩, 로드 밸런싱, 방화벽, [NAT](nat.md) 등 과거 별도 하드웨어(미들박스)로 구현하던 기능을 **단일 프로그래밍 추상화**로 통합
- 네트워크 동작을 소프트웨어로 정의 → 유연하고 빠른 혁신
- **P4**(Programming Protocol-independent Packet Processors): 더 풍부한 프로그래밍 언어로 라인 레이트 패킷 처리

## 출처
- [kurose-ch4](../sources/kurose-ch4.md)
