---
tags: [source, kurose]
source: "Kurose & Ross, Computer Networking: A Top-Down Approach, 8th Edition, Chapter 6"
topics: [link-layer, error-detection, CRC, multiple-access, ALOHA, CSMA, CSMA-CD, Ethernet, MAC-address, ARP, link-layer-switch, VLAN, MPLS, data-center-networking]
wiki_pages: [layers/link, protocols/ethernet, protocols/arp, concepts/mac-address, concepts/error-detection-correction, concepts/multiple-access, concepts/link-layer-switching, concepts/vlan, concepts/mpls, concepts/data-center-networking]
last_updated: 2026-04-08
---

# Kurose 8판 6장: The Link Layer and LANs

## 핵심 내용

### 6.1 [링크 계층](../layers/link.md) 소개
- 노드(Node)와 링크(Link) 용어 정의
- 링크 계층 서비스: 프레이밍, 링크 접근, 신뢰적 전달, [오류 검출/정정](../concepts/error-detection-correction.md)
- 구현 위치: NIC(네트워크 어댑터) 하드웨어 + 호스트 CPU 소프트웨어
- 링크의 두 유형: 점대점(point-to-point) vs 브로드캐스트(broadcast)

### 6.2 [오류 검출 및 정정 기법](../concepts/error-detection-correction.md)
- 패리티 검사: 1차원(검출만), 2차원(검출+정정)
- 체크섬: 인터넷 체크섬(16비트, [TCP](../protocols/tcp.md)/[UDP](../protocols/udp.md)/[IP](../protocols/ipv4.md)), 소프트웨어 구현에 적합
- CRC(순환 중복 검사): 다항식 코드, modulo-2 나눗셈, 링크 계층 하드웨어 구현
- FEC(전방 오류 정정): 재전송 없이 수신측에서 오류 정정

### 6.3 [다중 접속 프로토콜](../concepts/multiple-access.md)
- **채널 분할**: TDM, FDM, CDMA
- **랜덤 접속**: Slotted ALOHA(효율 1/e≈37%), Pure ALOHA(1/2e≈18%), CSMA, CSMA/CD
- CSMA/CD의 이진 지수 백오프: n번째 충돌 후 K ∈ {0,...,2^n-1}, 대기 = K×512 bit times
- CSMA/CD 효율 ≈ 1/(1+5d_prop/d_trans)
- **순번**: 폴링(마스터-슬레이브), 토큰 패싱(분산형)
- DOCSIS: FDM(다운) + TDM+랜덤(업) 혼합

### 6.4 스위치 LAN
- **[MAC 주소](../concepts/mac-address.md)**: 6바이트, 평면적, IEEE 관리, 브로드캐스트=FF-FF-FF-FF-FF-FF
- **[ARP](../protocols/arp.md)**: IP→MAC 변환, 브로드캐스트 질의/유니캐스트 응답, TTL 20분, 같은 서브넷 내에서만
- 서브넷 간 전송: 프레임의 MAC 주소는 매 홉마다 변경, IP 주소는 불변
- **[이더넷](../protocols/ethernet.md)**: 프레임 구조(Preamble+Dest+Src+Type+Data+CRC), 비연결/비신뢰, CSMA/CD
- 이더넷 진화: 버스→허브 스타→스위치 스타, 10M→100M→1G→10G→40G
- **[링크 계층 스위치](../concepts/link-layer-switching.md)**: self-learning, filtering/forwarding, plug-and-play, switch table
- 스위치 vs 라우터: L2 vs L3, 브로드캐스트 스톰 취약 vs 최적 경로
- **[VLAN](../concepts/vlan.md)**: 포트 기반 논리적 LAN 분할, 802.1Q 트렁킹(4바이트 태그, 12비트 VLAN ID)

### 6.5 링크 가상화: [MPLS](../concepts/mpls.md)
- MPLS 헤더: [Ethernet](../protocols/ethernet.md)과 IP 사이, 20비트 라벨+Exp+S+TTL
- 라벨 스위치 라우터: 라벨 기반 포워딩 ([IP 주소](../concepts/ip-addressing.md) 참조 불필요)
- 핵심 가치: 트래픽 엔지니어링(다중 경로, 부하 분산), 빠른 장애 복구, VPN

### 6.6 [데이터 센터 네트워킹](../concepts/data-center-networking.md)
- 구조: 블레이드→랙→TOR→Tier-2→Tier-1→Access Router→Border Router
- 로드 밸런서: [NAT](../concepts/nat.md) 유사 기능, Layer-4 스위치
- 대역폭 문제: 계층 상위로 갈수록 공유 → 호스트 간 대역폭 감소
- 해결: Clos 네트워크(고밀도 상호 연결), ECMP
- 트렌드: 커머디티 스위치, [SDN](../concepts/sdn-control-plane.md) 중앙 제어, VM 가상화, RDMA, MDC

### 6.7 웹 페이지 요청의 하루
- [DHCP](../protocols/dhcp.md)→[ARP](../protocols/arp.md)→[DNS](../protocols/dns.md)→[TCP](../protocols/tcp.md) 3-way handshake→[HTTP](../protocols/http.md) GET/응답
- [프로토콜 스택](../concepts/protocol-layers.md) 전체를 관통하는 통합 시나리오 (24단계)

## 위키에 반영된 페이지

- [link](../layers/link.md) — 링크 계층 개관
- [Ethernet](../protocols/ethernet.md) — 이더넷 프로토콜
- [ARP](../protocols/arp.md) — ARP 프로토콜
- [mac-address](../concepts/mac-address.md) — MAC 주소
- [error-detection-correction](../concepts/error-detection-correction.md) — 오류 검출/정정
- [multiple-access](../concepts/multiple-access.md) — 다중 접속 프로토콜
- [link-layer-switching](../concepts/link-layer-switching.md) — 링크 계층 스위칭
- [vlan](../concepts/vlan.md) — VLAN
- [mpls](../concepts/mpls.md) — MPLS
- [data-center-networking](../concepts/data-center-networking.md) — 데이터 센터 네트워킹
