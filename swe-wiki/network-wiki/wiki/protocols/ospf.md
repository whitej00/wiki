---
tags: [protocol, network-layer, control-plane, intra-AS]
layer: network
rfc: 2328
related: [routing-algorithms, BGP, ip-addressing, sdn-control-plane]
source_count: 1
last_updated: 2026-04-08
---

# OSPF (Open Shortest Path First)

인터넷에서 가장 널리 사용되는 **intra-AS(자율 시스템 내부) 라우팅 프로토콜**. [Link-State 알고리즘](../concepts/routing-algorithms.md)(Dijkstra)을 기반으로 하며, IS-IS와 함께 intra-AS 라우팅의 양대 산맥이다.

## 한 줄 요약

AS 내 모든 라우터가 link-state 정보를 브로드캐스트하여 완전한 토폴로지 맵을 구성하고, Dijkstra 알고리즘으로 최단 경로 트리를 계산한다.

## 왜 intra-AS 라우팅이 필요한가

인터넷의 라우터 수는 수억 대에 달하며, 하나의 라우팅 알고리즘으로 전체를 관리하는 것은 불가능하다:
- **규모(Scale)**: 라우팅 정보 저장·교환·계산 비용이 과도
- **관리 자율성(Administrative Autonomy)**: 각 ISP는 내부 라우팅을 독립적으로 운영하고 싶어 함

이를 해결하기 위해 라우터를 **자율 시스템(Autonomous System, AS)**으로 조직한다. 각 AS는 글로벌 고유 **ASN(Autonomous System Number)**으로 식별되며, AS 내부에서는 같은 라우팅 알고리즘을 실행한다.

## 동작 원리

1. 각 라우터가 직접 연결된 링크의 상태(비용, up/down)를 **link-state advertisement**로 AS 내 **모든** 라우터에 브로드캐스트
2. 각 라우터가 수신한 정보로 AS 전체의 토폴로지 그래프를 구성
3. 자신을 루트로 **Dijkstra 최단 경로 알고리즘**을 실행하여 모든 서브넷까지의 최단 경로 트리 계산
4. 최단 경로 트리에서 포워딩 테이블 구성

## 프로토콜 특성

| 항목 | 내용 |
|---|---|
| **전송** | IP 직접 전송 (upper-layer protocol 번호 89) |
| **링크 확인** | HELLO 메시지로 이웃 링크 상태 점검 |
| **주기적 갱신** | 변경 없어도 30분마다 link-state 브로드캐스트 (robustness 확보) |
| **신뢰성** | OSPF 자체적으로 신뢰적 메시지 전송 구현 |
| **링크 가중치** | 관리자 설정. 모두 1(최소 홉), 용량 역비례 등 자유 |

## 고급 기능

### 보안 (Security)
OSPF 패킷은 인증 가능. 두 가지 인증 방식:
- **Simple authentication**: 라우터에 같은 비밀번호 설정 (평문 전송, 보안 취약)
- **MD5 authentication**: 공유 비밀키로 패킷의 MD5 해시 계산·검증. 시퀀스 번호로 재전송 공격 방어

### 다중 동일비용 경로 (Multiple Same-cost Paths)
같은 비용의 경로가 여러 개 있으면 **복수 경로를 동시 사용** 가능. 단일 경로 선택 불필요.

### 멀티캐스트 (MOSPF)
Multicast OSPF [RFC 1584]. 기존 OSPF link-state DB에 새로운 link-state advertisement 타입을 추가하여 멀티캐스트 라우팅 지원.

### 계층적 영역 (Hierarchical Areas)
대규모 AS를 **영역(area)**으로 분할:
- 각 영역은 자체 OSPF link-state 알고리즘 실행
- 영역 내부 라우팅 정보는 영역 외부로 나가지 않음
- **영역 경계 라우터(Area Border Router)**: 영역 간 라우팅 담당
- **백본 영역(Backbone Area)**: AS당 정확히 하나. 모든 영역 경계 라우터를 포함하며 영역 간 트래픽을 중계
- 영역 간 라우팅: 출발 영역 → 경계 라우터 → 백본 → 목적 영역 경계 라우터 → 목적지

## 링크 가중치와 트래픽 엔지니어링

OSPF에서 링크 가중치는 관리자가 설정하며, Dijkstra 알고리즘이 이에 따라 경로를 결정한다. 실무에서는 인과관계가 **역전**되기도 한다: 원하는 트래픽 분배가 먼저 주어지고, 이를 달성하는 링크 가중치를 역산하는 것이 트래픽 엔지니어링(Traffic Engineering)의 핵심 과제다.

## 관련 프로토콜

- **IS-IS**: OSPF와 매우 유사한 LS 기반 intra-AS 프로토콜
- **EIGRP**: Cisco 독점(최근 공개), DV 계열
- [BGP](bgp.md): inter-AS 라우팅 프로토콜 (AS 간)

## 출처
- [kurose-ch5](../sources/kurose-ch5.md)
