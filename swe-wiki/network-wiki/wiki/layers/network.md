---
tags: [layer, network]
layer: network
related: [transport, link, IPv4, IPv6, forwarding-routing, router-architecture, routing-algorithms, OSPF, BGP, sdn-control-plane, ICMP, network-management, ipsec, firewall]
source_count: 3
last_updated: 2026-04-09
---

# 네트워크 계층 (Network Layer)

프로토콜 스택에서 가장 복잡한 계층. **모든 호스트와 라우터**에 존재하며, 송신 호스트에서 수신 호스트까지 데이터그램을 전달하는 **호스트 간(host-to-host) 통신**을 담당한다.

## 역할

송신 호스트의 전송 계층에서 세그먼트를 받아 데이터그램으로 캡슐화하고, 수신 호스트의 네트워크 계층까지 전달한다. 중간 라우터들은 네트워크 계층까지만 구현하며(전송·애플리케이션 계층 없음), 데이터그램의 헤더 필드를 검사하여 포워딩한다.

## Data Plane과 Control Plane

네트워크 계층은 두 가지 상호작용하는 부분으로 분해된다:

| | Data Plane | Control Plane |
|---|-----------|---------------|
| **기능** | [포워딩](../concepts/forwarding-routing.md) — 입력→출력 포트 전달 | [라우팅](../concepts/forwarding-routing.md) — 종단간 경로 결정 |
| **범위** | 라우터 로컬 | 네트워크 전체 |
| **시간 척도** | 나노초 | 초 |
| **구현** | 하드웨어 | 소프트웨어 |
| **Chapter** | Ch.4 | Ch.5 |

## 네트워크 서비스 모델

인터넷의 네트워크 계층은 **최선형 서비스(Best-effort Service)**를 제공한다:
- 전달 보장 없음
- 순서 보장 없음
- 지연 보장 없음
- 대역폭 보장 없음

"best-effort"는 사실상 "no service at all"에 가깝지만, 충분한 대역폭 확보와 DASH 같은 적응형 프로토콜 덕분에 대부분의 애플리케이션에 충분히 효과적이다.

다른 네트워크 아키텍처(예: ATM)는 보장된 전달, 제한된 지연, 최소 대역폭 등을 제공했으나, 인터넷의 best-effort 모델이 지배적이 되었다.

## 핵심 개념

### Data Plane (Ch.4)
- [forwarding-routing](../concepts/forwarding-routing.md) — 포워딩과 라우팅의 구분, data/control plane
- [router-architecture](../concepts/router-architecture.md) — 라우터 내부 구조
- [ip-addressing](../concepts/ip-addressing.md) — IP 주소 체계 (서브넷, CIDR)
- [packet-scheduling](../concepts/packet-scheduling.md) — 패킷 스케줄링 알고리즘
- [generalized-forwarding](../concepts/generalized-forwarding.md) — SDN과 OpenFlow 데이터 플레인
- [nat](../concepts/nat.md) — 네트워크 주소 변환
- [middleboxes](../concepts/middleboxes.md) — 미들박스와 NFV

### Control Plane (Ch.5)
- [routing-algorithms](../concepts/routing-algorithms.md) — 라우팅 알고리즘 (LS/Dijkstra, DV/Bellman-Ford)
- [sdn-control-plane](../concepts/sdn-control-plane.md) — SDN 컨트롤 플레인 (컨트롤러, OpenFlow 프로토콜, OpenDaylight/ONOS)
- [network-management](../concepts/network-management.md) — 네트워크 관리 (SNMP/MIB, NETCONF/YANG)

## 포함 프로토콜

- [IPv4](../protocols/ipv4.md) — 인터넷의 핵심 네트워크 계층 프로토콜 (32비트 주소)
- [IPv6](../protocols/ipv6.md) — 차세대 IP (128비트 주소)
- [DHCP](../protocols/dhcp.md) — 호스트에 IP 주소 자동 할당
- [ICMP](../protocols/icmp.md) — 오류 보고·진단 (ping, traceroute)
- [OSPF](../protocols/ospf.md) — Intra-AS 라우팅 (LS 기반)
- [BGP](../protocols/bgp.md) — Inter-AS 라우팅 (인터넷의 접착제)

## IP 모래시계 (IP Hourglass)

인터넷 프로토콜 스택에서 네트워크 계층에는 **IP 단 하나의 프로토콜**만 존재한다. 이 "좁은 허리(narrow waist)" 구조가 다양한 상·하위 계층 기술을 연결하는 인터넷 성장의 핵심이다.

```
      HTTP  SMTP  RTP  QUIC  DASH ...
            TCP        UDP
                 IP                ← narrow waist
      Ethernet  PPP  WiFi  Bluetooth ...
         copper   radio   fiber
```

## 컨트롤 플레인의 두 가지 접근

### Per-router Control (전통 방식)
각 라우터에서 라우팅 알고리즘이 실행되며, 라우터 간 라우팅 프로토콜(OSPF, BGP)을 통해 포워딩 테이블을 계산한다. 수십 년간 인터넷에서 사용되어 온 방식이다.

### Logically Centralized Control (SDN)
별도의 원격 컨트롤러가 포워딩 테이블/플로우 테이블을 계산하여 각 라우터의 Control Agent(CA)에 배포한다. CA는 컨트롤러와만 통신하며, 다른 CA와 직접 상호작용하지 않는다. Google B4 등 실제 대규모 배포 사례가 있다.

## 인터넷 라우팅의 계층 구조

```
    ┌────────────────────────────┐
    │   Inter-AS: BGP            │ ← AS 간 라우팅 (정책 우선)
    ├────────────────────────────┤
    │   Intra-AS: OSPF / IS-IS   │ ← AS 내부 라우팅 (성능 우선)
    └────────────────────────────┘
```

- **AS(Autonomous System)**: 같은 관리 하의 라우터 그룹. ASN으로 식별.
- intra-AS와 inter-AS를 분리하는 이유: **규모**(확장성)와 **관리 자율성**(정책 독립)

## 네트워크 계층 보안

- [IPsec](../protocols/ipsec.md) — IP 데이터그램 수준에서 기밀성, 출처 인증, 무결성, 재생 공격 방지를 제공 ("blanket coverage"). VPN의 핵심 기술
- [firewall](../concepts/firewall.md) — 조직 네트워크와 인터넷 사이에서 패킷 필터링 (전통적/상태기반/애플리케이션 게이트웨이)
- [ids-ips](../concepts/ids-ips.md) — 심층 패킷 검사로 침입 탐지/방지 (시그니처 기반/이상 기반)

## 출처
- [kurose-ch4](../sources/kurose-ch4.md)
- [kurose-ch5](../sources/kurose-ch5.md)
- [kurose-ch8](../sources/kurose-ch8.md)
