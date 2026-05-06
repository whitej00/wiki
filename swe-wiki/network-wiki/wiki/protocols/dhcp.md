---
tags: [protocol, network-layer, application-layer]
layer: application
rfc: 2131
related: [IPv4, ip-addressing, NAT, UDP, DNS]
source_count: 1
last_updated: 2026-04-08
---

# DHCP (Dynamic Host Configuration Protocol)

RFC 2131에 정의된 클라이언트-서버 프로토콜. 호스트가 네트워크에 접속할 때 **IP 주소, 서브넷 마스크, 기본 게이트웨이, DNS 서버 주소** 등을 자동으로 할당받을 수 있게 해주는 **plug-and-play(zeroconf)** 프로토콜이다. [UDP](udp.md) 위에서 동작한다.

## 왜 필요한지

IP 주소를 수동으로 설정하는 것은 특히 사용자가 자주 변경되는 환경(학교, 카페, 기업 네트워크 등)에서 비현실적이다. DHCP는 호스트가 네트워크에 연결되는 순간 자동으로 네트워크 설정을 완료한다.

## 4단계 동작

### 1. DHCP Discover (클라이언트 → 브로드캐스트)
- 새로 접속한 호스트가 DHCP 서버를 찾기 위해 전송
- **UDP 포트 67**로 전송
- 출발지 IP: **0.0.0.0** (아직 IP 없음)
- 목적지 IP: **255.255.255.255** (브로드캐스트)
- 트랜잭션 ID 포함

### 2. DHCP Offer (서버 → 브로드캐스트)
- 서버가 제안 IP 주소, 서브넷 마스크, **임대 시간(lease time)** 등을 포함하여 응답
- 여러 DHCP 서버가 존재할 수 있으므로 여러 offer를 받을 수 있음
- 목적지 IP: 255.255.255.255 (클라이언트가 아직 IP를 갖지 않으므로)

### 3. DHCP Request (클라이언트 → 브로드캐스트)
- 클라이언트가 offer 중 하나를 선택하여 요청
- 선택한 DHCP 서버 ID와 제안받은 IP 주소를 포함

### 4. DHCP ACK (서버 → 브로드캐스트)
- 서버가 요청을 확인하고 설정 파라미터 확정
- 이후 클라이언트는 임대 기간 동안 할당받은 IP 사용 가능

## 제공하는 정보

- **IP 주소**: 클라이언트에 할당될 주소
- **서브넷 마스크**: 로컬 네트워크 범위
- **기본 게이트웨이(Default Gateway)**: first-hop 라우터 주소
- **DNS 서버 주소**: 로컬 DNS 서버

## 임대 시간 (Lease Time)

IP 주소는 영구 할당이 아닌 **임대** 방식. 통상 수 시간~수 일. 임대 만료 전 갱신(renew) 가능.

## DHCP 릴레이 에이전트

모든 서브넷에 DHCP 서버가 있을 필요는 없다. **DHCP 릴레이 에이전트**(통상 라우터)가 클라이언트의 브로드캐스트를 다른 서브넷의 DHCP 서버에 유니캐스트로 중계한다.

## 한계

서브넷 이동 시 새 IP를 할당받으므로, 기존 TCP 연결이 끊어진다. 모바일 환경에서의 연결 유지는 별도 메커니즘 필요 (Ch.7).

## 출처
- [kurose-ch4](../sources/kurose-ch4.md)
