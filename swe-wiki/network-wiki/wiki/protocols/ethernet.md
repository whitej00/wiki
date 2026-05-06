---
tags: [protocol, link-layer]
layer: link
rfc:
related: [ARP, mac-address, multiple-access, link-layer-switching, vlan, CSMA-CD]
source_count: 1
last_updated: 2026-04-08
---

# 이더넷 (Ethernet)

유선 LAN 시장을 지배하는 **링크 계층 프로토콜**이자 물리 계층 사양. 1970년대 중반 Bob Metcalfe와 David Boggs가 Xerox PARC에서 발명했으며, IEEE 802.3 표준으로 규격화되었다.

## 이더넷의 성공 요인

1. **선점자 이점** — 최초로 널리 배포된 고속 LAN
2. **경쟁 기술 대비 단순·저렴** — Token Ring, FDDI, ATM보다 간단
3. **속도 경쟁력 유지** — 10 Mbps → 100 Mbps → 1 Gbps → 10 Gbps → 40 Gbps로 지속 진화
4. **프레임 포맷 불변** — 30년 이상 동일한 프레임 포맷 유지

## 토폴로지 진화

| 시대 | 토폴로지 | 특성 |
|------|----------|------|
| 1980s–1990s 초 | **버스(Bus)** — 동축 케이블 | 브로드캐스트 LAN, CSMA/CD 필수 |
| 1990s 후반 | **허브 기반 스타(Star)** | 허브가 모든 비트를 모든 포트로 전달, 여전히 브로드캐스트 |
| 2000s– | **스위치 기반 스타(Star)** | 스위치가 store-and-forward, **충돌 없음**, MAC 프로토콜 불필요 |

## 이더넷 프레임 구조

```
| Preamble (8B) | Dest Addr (6B) | Src Addr (6B) | Type (2B) | Data (46-1500B) | CRC (4B) |
```

| 필드 | 크기 | 설명 |
|------|------|------|
| **프리앰블(Preamble)** | 8바이트 | 처음 7바이트 `10101010`: 수신 어댑터 클록 동기화. 마지막 바이트 `10101011`: 프레임 시작 알림 |
| **목적지 주소(Dest Address)** | 6바이트 | 수신 어댑터의 [MAC 주소](../concepts/mac-address.md). 일치하거나 브로드캐스트면 상위 전달, 아니면 폐기 |
| **출발지 주소(Src Address)** | 6바이트 | 송신 어댑터의 MAC 주소 |
| **타입(Type)** | 2바이트 | 상위 네트워크 계층 프로토콜 식별 (IP: `0x0800`, [ARP](arp.md): `0x0806`). 역다중화 역할 |
| **데이터(Data)** | 46–1,500바이트 | IP 데이터그램 등 페이로드. MTU = 1,500바이트. 최소 46바이트 (부족 시 스터핑) |
| **CRC** | 4바이트 | [CRC-32](../concepts/error-detection-correction.md)로 비트 오류 검출 |

## 이더넷의 서비스 특성

- **비연결형(Connectionless)** — 핸드셰이크 없이 프레임 전송 (IP, UDP와 유사)
- **비신뢰적(Unreliable)** — CRC 실패 시 프레임 폐기, ACK/NAK 없음. 상위 계층(TCP)이 재전송 담당
- **CSMA/CD** — 전통적 이더넷의 [매체 접근 제어](../concepts/multiple-access.md) 프로토콜. 현대 스위치 기반 이더넷에서는 충돌이 없으므로 실질적으로 불필요

## 이더넷 기술 표준

표준 명명 규칙: `{속도}BASE-{매체}` (예: 100BASE-T)
- 속도: 10, 100, 1000, 10G, 40G
- BASE: 베이스밴드 전송
- 매체: T(twisted-pair copper), FX/SX/LX/BX(광섬유)

| 표준 | 속도 | 매체 |
|------|------|------|
| 10BASE-T | 10 Mbps | UTP |
| 100BASE-T | 100 Mbps | UTP |
| 1000BASE-T | 1 Gbps | Cat 5 UTP |
| 10GBASE-T | 10 Gbps | Cat 6a UTP |
| 100BASE-FX | 100 Mbps | 광섬유 |
| 1000BASE-LX | 1 Gbps | 광섬유(장거리) |

기가비트 이더넷(IEEE 802.3z)은 점대점 채널에서 **40 Gbps 전이중(full-duplex)** 동작을 지원한다.

## 출처

- [kurose-ch6](../sources/kurose-ch6.md) — Kurose 8판 6장 (Section 6.4.2)
