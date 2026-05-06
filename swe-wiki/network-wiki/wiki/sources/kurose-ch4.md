---
tags: [source, kurose, network-layer]
source_type: textbook
chapter: 4
pages: 333-406
last_updated: 2026-04-08
---

# Kurose 8판 Chapter 4: The Network Layer — Data Plane

## 소스 정보
- **교재**: Computer Networking: A Top-Down Approach, 8th Edition
- **저자**: James F. Kurose, Keith W. Ross
- **챕터**: 4 — The Network Layer: Data Plane
- **페이지**: pp. 333–406

## 핵심 내용

### 4.1 [네트워크 계층](../layers/network.md) 개관
- 네트워크 계층은 **모든 호스트와 라우터**에 존재 ([전송](../layers/transport.md)/[애플리케이션](../layers/application.md) 계층과 달리)
- 두 가지 핵심 기능: **[포워딩(Forwarding, data plane)과 라우팅(Routing, control plane)](../concepts/forwarding-routing.md)**
- 포워딩: 라우터 로컬 동작, 입력→출력 포트 전달 (나노초, 하드웨어)
- 라우팅: 네트워크 전체 경로 결정 (초 단위, 소프트웨어)
- **포워딩 테이블**: 라우터의 핵심 데이터 구조
- Control plane 접근: (1) 전통적 — 각 라우터에서 라우팅 알고리즘 실행, (2) [SDN](../concepts/sdn-control-plane.md) — 원격 컨트롤러가 포워딩 테이블 계산·배포
- **SDN(Software-Defined Networking)**: 컨트롤러가 데이터 플레인과 분리된 제어 플레인 구현
- **네트워크 서비스 모델**: 인터넷은 **best-effort** 서비스 — 전달 보장, 순서 보장, 지연 보장, 대역폭 보장 모두 없음. ATM은 보장 서비스 제공했으나 인터넷의 best-effort가 성공

### 4.2 [라우터 내부](../concepts/router-architecture.md)
- 4대 구성요소: **입력 포트**, **스위칭 패브릭**, **출력 포트**, **라우팅 프로세서**
- 데이터 플레인(입력/출력 포트, 스위칭 패브릭): 하드웨어, 나노초 단위
- 제어 플레인(라우팅 프로세서): 소프트웨어, 밀리초/초 단위

#### 입력 포트와 목적지 기반 포워딩
- 라인 종단 → 링크 계층 처리 → **룩업**(포워딩 테이블 조회) → 스위칭 패브릭
- **최장 프리픽스 매칭(Longest Prefix Matching)**: 여러 엔트리 매칭 시 가장 긴 프리픽스 선택
- TCAM(Ternary Content Addressable Memory)으로 상수 시간 룩업

#### 스위칭 패브릭
- **메모리 기반**: 라우팅 프로세서가 패킷을 메모리 복사. 속도 = 메모리 대역폭/2
- **버스 기반**: 입력→출력을 공유 버스로 직접 전달. 한 번에 하나만 가능
- **교차바(Crossbar) 네트워크**: 2N 버스, 병렬 전달 가능, **non-blocking**

#### 출력 포트
- 큐잉(버퍼) 관리 → 링크 계층 처리 → 라인 종단

#### 큐잉
- **입력 큐잉**: 스위칭 패브릭이 느릴 때 발생. **HOL(Head-of-Line) 블로킹** — 큐 앞 패킷이 막히면 뒤 패킷도 대기 (출력 포트가 비어있어도)
- **출력 큐잉**: 여러 입력에서 같은 출력으로 집중 시 발생. 메모리 소진 시 패킷 손실
- **drop-tail**: 도착 패킷 폐기. 또는 기존 패킷 제거(AQM)
- **AQM(Active Queue Management)**: RED, PIE, CoDel 등
- **버퍼 크기**: 경험적 공식 B = RTT · C (소수 TCP 흐름), B = RTT · C / √N (다수 TCP 흐름)
- **Bufferbloat**: 과도한 버퍼로 인한 지속적 큐잉 지연. DOCSIS 3.1이 AQM 도입

#### [패킷 스케줄링](../concepts/packet-scheduling.md)
- **FIFO**: 도착 순서대로 전송
- **Priority Queuing**: 우선순위 클래스별 큐. 높은 클래스 우선. non-preemptive
- **Round Robin (RR)**: 클래스 간 교대 서비스
- **WFQ(Weighted Fair Queuing)**: RR에 가중치 적용. 클래스 i의 보장 처리량 ≥ R · w_i / Σw_j
- **망 중립성(Net Neutrality)**: No Blocking, No Throttling, No Paid Prioritization (FCC 2015). 2017 FCC에서 완화

### 4.3 IP: [IPv4](../protocols/ipv4.md), [주소](../concepts/ip-addressing.md), [IPv6](../protocols/ipv6.md)
#### [IPv4](../protocols/ipv4.md) 데이터그램
- **20바이트 헤더** (옵션 없을 때)
- 주요 필드: Version(4), Header length(4), TOS(8, ECN 포함), Datagram length(16), Identifier/Flags/Fragment offset(단편화), **TTL**(8), Protocol(8, TCP=6/UDP=17), Header checksum(16), Source/Destination IP(각 32비트), Options, Data
- 단편화: IPv4에서 가능 (목적지에서 재조립). IPv6에서는 금지

#### [IPv4 주소](../concepts/ip-addressing.md)
- **인터페이스** 단위 할당 (호스트/라우터가 아님)
- 32비트, **점-10진 표기(Dotted-decimal notation)**
- **서브넷(Subnet)**: 라우터 없이 직접 연결된 인터페이스의 네트워크. /24 등으로 표기
- **CIDR(Classless Interdomain Routing)**: a.b.c.d/x — x비트가 네트워크 프리픽스
- **Classful 주소**: Class A(/8), B(/16), C(/24) — 유연성 부족으로 CIDR로 대체
- **경로 집약(Route Aggregation)**: ISP가 하위 조직들을 단일 프리픽스로 광고
- **브로드캐스트 주소**: 255.255.255.255
- 주소 블록 할당: ICANN → 지역 레지스트리(ARIN, RIPE, APNIC 등) → ISP → 조직

#### [DHCP](../protocols/dhcp.md) (Dynamic Host Configuration Protocol)
- RFC 2131. **plug-and-play/zeroconf** 프로토콜
- 4단계: (1) DHCP discover (브로드캐스트), (2) DHCP offer, (3) DHCP request, (4) DHCP ACK
- IP 주소 외에 서브넷 마스크, 기본 게이트웨이, DNS 서버 주소도 제공
- **임대 시간(Lease time)**: 주소 사용 기간 제한

#### [NAT](../concepts/nat.md) (Network Address Translation)
- RFC 2663, 3022. 사설 네트워크(10.0.0.0/8 등) 내부 주소를 단일 공인 IP로 변환
- **NAT 변환 테이블**: (WAN측 IP:포트) ↔ (LAN측 IP:포트) 매핑
- 단일 공인 IP로 ~60,000개 동시 연결 지원 (16비트 포트)
- 비판: 포트는 프로세스 식별용이지 호스트 식별용이 아님, 계층 원칙 위반
- **NAT traversal**: RFC 5389, 5128 등으로 NAT 뒤 서버 접근 문제 해결

#### 방화벽과 IDS
- **방화벽**: 헤더 필드 기반 패킷 필터링, TCP 연결 추적
- **IDS(Intrusion Detection System)**: DPI(심층 패킷 검사), 시그니처 매칭
- **IPS**: IDS + 차단 기능

#### [IPv6](../protocols/ipv6.md)
- RFC 2460. **128비트 주소**, **40바이트 고정 헤더**
- 주요 변경: 단편화 제거(라우터에서), 헤더 체크섬 제거, 옵션 필드를 next header로 대체
- 필드: Version(4), Traffic class(8), **Flow label(20)**, Payload length(16), Next header(8), **Hop limit(8)**, Source/Destination address(각 128비트)
- **Anycast 주소**: 그룹 내 가장 가까운 호스트에 전달
- **IPv4→IPv6 전환**: **터널링(Tunneling)** — IPv6 데이터그램을 IPv4 데이터그램의 페이로드에 캡슐화
- IPv6 채택 가속 중 (Google 서비스 ~25% IPv6, 3GPP가 모바일 멀티미디어에 IPv6 표준화)

### 4.4 [일반화된 포워딩](../concepts/generalized-forwarding.md)과 SDN
- 목적지 기반 포워딩의 일반화: **match-plus-action** 패러다임
- **OpenFlow**: SDN의 핵심 표준. 플로우 테이블의 각 엔트리 = 매칭 필드 + 카운터 + 액션
- **매칭 필드**: 링크 계층(MAC), 네트워크 계층(IP), 전송 계층(포트) 등 11개+ 필드
- **액션**: 포워딩, 드롭, 필드 수정
- 예시: 단순 포워딩, 로드 밸런싱, 방화벽 — 모두 플로우 테이블로 구현 가능
- **P4**: 프로토콜 독립 패킷 프로세서 프로그래밍 언어

### 4.5 [미들박스](../concepts/middleboxes.md)
- RFC 3234 정의: "소스와 목적지 사이 데이터 경로에서 표준 IP 라우터 기능 외의 기능을 수행하는 중간 장치"
- 3가지 유형: (1) [NAT](../concepts/nat.md), (2) 보안 서비스(방화벽/IDS), (3) 성능 향상(압축/캐싱/로드밸런싱)
- **NFV(Network Function Virtualization)**: [SDN](../concepts/sdn-control-plane.md) 접근으로 미들박스 기능을 소프트웨어로 구현
- **IP 모래시계(IP Hourglass)**: 네트워크 계층에 IP 하나만 존재 — 인터넷 성장의 핵심
- **[End-to-end 논거](../concepts/end-to-end-principle.md)**: 지능은 네트워크 코어가 아닌 종단에 배치 [Saltzer 1984, RFC 1958]

## 다룬 토픽 목록
- [forwarding-routing](../concepts/forwarding-routing.md), [router-architecture](../concepts/router-architecture.md), [ip-addressing](../concepts/ip-addressing.md), [IPv4](../protocols/ipv4.md), [IPv6](../protocols/ipv6.md), [DHCP](../protocols/dhcp.md), [NAT](../concepts/nat.md), [packet-scheduling](../concepts/packet-scheduling.md), [generalized-forwarding](../concepts/generalized-forwarding.md), [middleboxes](../concepts/middleboxes.md), [network](../layers/network.md)

## 위키에 반영된 페이지
- protocols/ipv4.md, protocols/ipv6.md, protocols/dhcp.md (신규)
- concepts/forwarding-routing.md, concepts/router-architecture.md, concepts/ip-addressing.md, concepts/nat.md, concepts/packet-scheduling.md, concepts/generalized-forwarding.md, concepts/middleboxes.md (신규)
- layers/network.md (신규)
