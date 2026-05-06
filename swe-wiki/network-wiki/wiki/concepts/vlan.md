---
tags: [concept, link-layer]
layer: link
related: [link-layer-switching, Ethernet, mac-address]
source_count: 1
last_updated: 2026-04-08
---

# VLAN (Virtual Local Area Network)

하나의 물리적 스위치 인프라 위에 **여러 개의 논리적 LAN**을 정의할 수 있게 하는 기술. VLAN 내의 호스트는 같은 스위치에만 연결된 것처럼 통신한다.

## 왜 필요한가

단순한 스위치 계층 구조에서 발생하는 세 가지 문제:

1. **트래픽 격리 부재** — 브로드캐스트 트래픽(ARP, DHCP 등)이 전체 네트워크로 전파. 보안/프라이버시 문제
2. **스위치 비효율** — 그룹이 많으면 그룹 수만큼 스위치 필요, 소규모 그룹에서는 스위치 포트 낭비
3. **사용자 관리 어려움** — 부서 이동 시 물리적 케이블 재배선 필요

## 포트 기반 VLAN

스위치의 포트(인터페이스)를 **그룹으로 분할**하여 각 그룹이 독립적인 브로드캐스트 도메인을 형성.

예: 16포트 스위치에서
- 포트 2–8: EE VLAN
- 포트 9–15: CS VLAN
- 포트 1, 16: 미할당

각 VLAN은 독립된 브로드캐스트 도메인으로 동작하므로, EE VLAN의 브로드캐스트 프레임은 CS VLAN에 도달하지 않는다.

## VLAN 간 통신

완전히 격리된 VLAN 간에 트래픽을 전달하려면 **라우터**가 필요하다:
- VLAN 스위치 포트 하나를 라우터에 연결
- 실제로는 스위치 벤더가 **스위치 + 라우터 일체형** 장비를 제공하여 별도의 외부 라우터가 불필요

## VLAN 트렁킹 (VLAN Trunking)

여러 스위치에 걸쳐 동일한 VLAN을 구성할 때 사용.

### 문제
N개의 VLAN이 있으면, 스위치 간 VLAN별로 N개의 포트를 연결해야 함 → 확장 불가

### 해결: 트렁크 포트
- 스위치 간 **하나의 트렁크 링크**로 모든 VLAN의 프레임을 전달
- 트렁크 포트는 **모든 VLAN에 소속**

### IEEE 802.1Q

트렁크 링크에서 프레임이 어떤 VLAN에 속하는지 식별하기 위한 표준.

기존 이더넷 프레임에 **4바이트 VLAN 태그**를 추가:

```
| Preamble | Dest | Src | Tag Protocol ID (2B) | Tag Control Info (2B) | Type | Data | CRC' |
```

| 필드 | 크기 | 설명 |
|------|------|------|
| TPID (Tag Protocol Identifier) | 2바이트 | 고정값 `0x8100` — 802.1Q 태그 프레임임을 표시 |
| TCI (Tag Control Information) | 2바이트 | **VLAN ID** (12비트, 최대 4,096개 VLAN) + 우선순위(3비트) |

- VLAN 태그는 송신 측 스위치가 트렁크로 보낼 때 삽입하고, 수신 측 스위치가 제거
- CRC는 태그 추가 후 **재계산**

## 기타 VLAN 유형

- **MAC 기반 VLAN** — MAC 주소로 VLAN 자동 배정
- **네트워크 프로토콜 기반 VLAN** — IPv4, IPv6, AppleTalk 등 프로토콜별 분류
- **IP 라우터를 통한 VLAN 확장** — 지리적으로 분산된 VLAN을 하나로 연결 가능

## 출처

- [kurose-ch6](../sources/kurose-ch6.md) — Kurose 8판 6장 (Section 6.4.4)
