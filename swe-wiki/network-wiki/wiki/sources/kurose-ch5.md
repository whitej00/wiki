---
tags: [source, kurose, network-layer, control-plane]
book: "Computer Networking: A Top-Down Approach, 8th Edition"
authors: [James F. Kurose, Keith W. Ross]
chapter: 5
title: "The Network Layer: Control Plane"
pages: 407-478
source_count: 1
last_updated: 2026-04-08
---

# Kurose 8판 5장: The Network Layer — Control Plane

Ch.4(데이터 플레인)에 이어 네트워크 계층의 **컨트롤 플레인**을 다룬다. 포워딩 테이블/플로우 테이블이 어떻게 계산되고 설치되는지를 중심으로, 라우팅 알고리즘, 인터넷 라우팅 프로토콜(OSPF, BGP), SDN, ICMP, 네트워크 관리를 다룬다.

## 다룬 토픽

### 5.1 Introduction
- 컨트롤 플레인의 두 가지 접근: **per-router control**(각 라우터에서 라우팅 알고리즘 실행) vs **logically centralized control**(SDN 컨트롤러가 플로우 테이블 배포)
- per-router: 라우터 간 라우팅 프로토콜로 포워딩 테이블 계산
- SDN: 원격 컨트롤러가 Control Agent(CA)를 통해 플로우 테이블 관리

### 5.2 [Routing Algorithms](../concepts/routing-algorithms.md)
- 그래프 G = (N, E) 모델, 링크 비용 c(x,y), 최소 비용 경로 문제
- **분류**: centralized(LS) vs decentralized(DV) / static vs dynamic / load-sensitive vs load-insensitive
- 현대 인터넷 라우팅([OSPF](../protocols/ospf.md), [BGP](../protocols/bgp.md))은 load-insensitive

#### 5.2.1 Link-State (LS) — Dijkstra 알고리즘
- 전제: 모든 노드가 link-state broadcast로 전체 토폴로지 파악
- D(v): 소스→v 최소 비용, p(v): 이전 노드, N': 확정 노드 집합
- 복잡도 O(n^2), 힙 사용 시 O(n log n)
- **진동 문제**: 혼잡 기반 링크 비용 시 라우트 oscillation → 해결: 링크 비용을 부하 독립으로 하거나, LS 실행 시점 랜덤화

#### 5.2.2 Distance-Vector (DV) — Bellman-Ford
- d_x(y) = min_v { c(x,v) + d_v(y) } (Bellman-Ford 방정식)
- 분산/비동기/반복적, 자기종료형
- 각 노드: 자기 거리벡터 + 이웃 거리벡터 유지 → 링크 비용 변경 또는 이웃 DV 수신 시 재계산 → 변경 시 이웃에 전파
- **count-to-infinity 문제**: 링크 비용 증가 시 수렴 매우 느림 (44회 반복 예시)
- **poisoned reverse**: z가 y를 경유해 x에 도달하면 y에게 D_z(x)=∞ 보고 → 2노드 루프 해결, 3노드 이상 루프는 미해결

#### LS vs DV 비교
| | LS | DV |
|---|---|---|
| 메시지 복잡도 | O(\|N\|\|E\|) | 수렴 시간 가변 |
| 수렴 속도 | O(n^2) | 느림, 루프 가능 |
| 강건성 | 각 노드 독립 계산 | 오류 전파 가능 |

### 5.3 Intra-AS Routing: [OSPF](../protocols/ospf.md)
- **자율 시스템(AS)**: 같은 관리 하의 라우터 그룹, ASN으로 식별
- **intra-AS 라우팅 프로토콜**: AS 내부 라우팅
- [OSPF](../protocols/ospf.md): LS 기반, Dijkstra, RFC 2328, IP 직접 전송(프로토콜 번호 89)
- HELLO 메시지로 링크 상태 확인, 30분마다 주기적 브로드캐스트
- 고급 기능: 보안(MD5 인증), 다중 동일비용 경로, 멀티캐스트(MOSPF), 계층적 영역(area/backbone)
- 링크 가중치 설정은 관리자 재량 (트래픽 엔지니어링에 활용)

### 5.4 Routing Among ISPs: [BGP](../protocols/bgp.md)
- **inter-AS 라우팅 프로토콜**: AS 간 라우팅, 인터넷의 "접착제"
- CIDRized prefix 단위 목적지, [TCP](../protocols/tcp.md) 포트 179 세미영구 연결
- **eBGP**: AS 간 게이트웨이 라우터 연결 / **iBGP**: AS 내부 라우터 간 메시
- **BGP 속성**: AS-PATH(경유 AS 목록, 루프 탐지), NEXT-HOP(AS-PATH 시작 라우터 IP)
- **Hot potato routing**: 자기 AS 내 최소 비용 게이트웨이로 즉시 전달
- **경로 선택 알고리즘** (순서대로):
  1. 최고 local preference
  2. 최단 AS-PATH
  3. Hot potato (최근접 NEXT-HOP)
  4. BGP identifier로 타이브레이크
- **IP-Anycast**: 동일 IP를 여러 서버에서 광고 → BGP가 가장 가까운 서버로 라우팅 (DNS 루트 서버에 활용)
- **라우팅 정책**: provider/customer/peer 관계, 선택적 경로 광고로 트래픽 흐름 통제
- **인터넷 프레즌스 획득**: ISP 계약 → IP 할당 → DNS 등록 → BGP 광고

### 5.5 [SDN Control Plane](../concepts/sdn-control-plane.md)
- SDN 4대 특성: flow-based forwarding, data/control plane 분리, 외부 제어, 프로그래머블 네트워크
- **SDN 컨트롤러 3계층**: communication layer(southbound, OpenFlow) → state-management layer → application layer(northbound, REST API)
- 논리적으로 중앙집중, 물리적으로 분산 구현 (fault tolerance, scalability)
- **OpenFlow 프로토콜** (TCP 6653):
  - Controller→Switch: Configuration, Modify-State, Read-State, Send-Packet
  - Switch→Controller: Flow-Removed, Port-status, Packet-in
- **SDN 컨트롤러 사례**: OpenDaylight(ODL), ONOS — 모두 Linux Foundation 오픈소스
- **Google B4**: OpenFlow 기반 WAN SDN, 링크 이용률 70%+ 달성
- SDN 역사: Ethane(2007) → OpenFlow → 현재 NFV 등으로 확장

### 5.6 [ICMP](../protocols/icmp.md)
- IP 위에서 동작(프로토콜 번호 1), type/code 필드 + 오류 유발 IP 데이터그램 첫 8바이트
- 주요 메시지: echo request/reply(type 8/0, ping), destination unreachable(type 3), TTL expired(type 11, traceroute), source quench(type 4)
- **Traceroute**: TTL 1부터 증가하며 UDP 전송 → TTL 만료 ICMP(type 11)로 경로 추적 → 목적지 도착 시 port unreachable(type 3 code 3)으로 종료 감지
- ICMPv6 (RFC 4443): IPv6용 확장, "Packet Too Big" 등 추가

### 5.7 [Network Management](../concepts/network-management.md) and SNMP, NETCONF/YANG
- **관리 프레임워크 4요소**: managing server, managed device, data(config/operational/statistics), network management agent, protocol
- **3가지 관리 접근**: CLI / SNMP+MIB / NETCONF+YANG
- **SNMP** (RFC 3410): 요청-응답 + trap, UDP 전송, 7종 PDU(GetRequest, GetNextRequest, GetBulkRequest, SetRequest, InformRequest, Response, SNMPv2-Trap)
- **MIB**: SMI로 정의된 관리 객체, 400+ MIB 모듈
- **NETCONF** (RFC 6241): XML 기반 RPC, TLS/TCP 보안, 원자적 트랜잭션 지원
  - 오퍼레이션: get-config, get, edit-config, lock/unlock, create-subscription/notification
- **YANG** (RFC 6020): NETCONF 데이터 모델링 언어, 정확성/일관성 제약 명시

## 위키에 반영된 페이지
- [routing-algorithms](../concepts/routing-algorithms.md) — LS/DV 알고리즘, 분류, 비교
- [OSPF](../protocols/ospf.md) — Intra-AS 라우팅 프로토콜
- [BGP](../protocols/bgp.md) — Inter-AS 라우팅 프로토콜
- [sdn-control-plane](../concepts/sdn-control-plane.md) — SDN 컨트롤러, OpenFlow
- [ICMP](../protocols/icmp.md) — 에러 보고, traceroute
- [network-management](../concepts/network-management.md) — SNMP/MIB, NETCONF/YANG
- [network](../layers/network.md) — 네트워크 계층 (컨트롤 플레인 보강)
