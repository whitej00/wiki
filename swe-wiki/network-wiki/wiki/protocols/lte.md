---
tags: [protocol, network, link]
layer: link
standard: 3GPP
related: [cellular-network-architecture, mobility-management, wireless-link-characteristics, nat, IP addressing]
source_count: 1
last_updated: 2026-04-09
---

# 4G LTE (Long-Term Evolution)

**4G LTE**는 현재 가장 널리 보급된 셀룰러 네트워크 표준으로, **all-IP 패킷 교환** 아키텍처를 채택한다. 음성과 데이터를 모두 IP 기반으로 처리하며, 이전 세대(2G 회선교환, 3G 혼합)에서 진화한 것이다.

## 한 줄 요약

eNode-B(기지국), MME(인증/이동성 제어), S-GW/P-GW(데이터 경로), HSS(가입자 DB)로 구성된 all-IP 셀룰러 네트워크.

## 네트워크 아키텍처

### 주요 구성요소

| 요소 | 역할 | 플레인 |
|------|------|--------|
| **Mobile Device (UE)** | 스마트폰/태블릿/IoT 장치. 5계층 인터넷 프로토콜 스택 구현. IMSI 보유 | Data + Control |
| **Base Station (eNode-B)** | 무선 자원 관리, 장치 인증 조율, IP 터널 생성, 셀 간 간섭 관리 | Data + Control |
| **MME** (Mobility Management Entity) | 인증(HSS와 연동), 터널 설정, 위치 추적, paging | Control only |
| **HSS** (Home Subscriber Service) | 가입자 DB (IMSI, 인증 키, 서비스 정보, home network 정보) | Control |
| **S-GW** (Serving Gateway) | 방문 네트워크의 라우터, 기지국-코어 간 데이터 중계 | Data |
| **P-GW** (PDN Gateway) | 홈 네트워크의 게이트웨이 라우터, NAT 수행, 인터넷 연결 | Data |

### IMSI (International Mobile Subscriber Identity)

SIM 카드에 저장된 **64비트 글로벌 가입자 식별자**. 가입 국가, 홈 네트워크, 가입자를 유일하게 식별한다. MAC 주소와 유사한 역할이지만 글로벌하게 라우팅 가능하다.

### Data Plane vs Control Plane

- **Data plane**: Mobile Device ↔ Base Station ↔ S-GW ↔ P-GW ↔ Internet
  - 기지국-S-GW, S-GW-P-GW 사이에 **GTP**(GPRS Tunneling Protocol) 터널 2개 설정
  - **간접 라우팅(Indirect Routing)**: 모든 트래픽이 홈 네트워크의 P-GW를 경유
- **Control plane**: Base Station ↔ MME ↔ HSS
  - MME는 데이터 경로에 포함되지 않음

## LTE 프로토콜 스택

무선 링크(Mobile Device ↔ Base Station)에서 링크 계층을 세 개 서브레이어로 분할:

| 서브레이어 | 기능 |
|----------|------|
| **PDCP** (Packet Data Convergence) | IP 헤더 압축/해제, 암호화/복호화 |
| **RLC** (Radio Link Control) | IP 데이터그램 단편화/재조립, ARQ 기반 신뢰적 전송 |
| **MAC** | 전송 스케줄링, 라디오 자원 할당, FEC (forward error correction) |

기지국과 S-GW 이후는 일반 IP/UDP/GTP-U 스택을 사용한다.

## Radio Access Network

**OFDM**(Orthogonal Frequency Division Multiplexing)을 사용한 시분할 + 주파수 분할 결합:

- 다운스트림: 0.5 ms 타임슬롯, 복수 주파수 채널
- 각 모바일 장치에 하나 이상의 타임슬롯 + 주파수 채널 할당
- 슬롯 재할당은 1 ms마다 가능 → **opportunistic scheduling** (채널 상태에 따라 최적 사용자에게 할당)
- LTE-Advanced: 채널 어그리게이션으로 수백 Mbps 달성

## Network Attachment (네트워크 접속)

3단계 과정:

1. **기지국 탐색/연관**: 모든 주파수 대역에서 primary/secondary synchronization signal 탐색 → 기지국 선택 → control-plane signaling 채널 수립 → IMSI 제공
2. **상호 인증**: 기지국 → 로컬 MME → HSS. 네트워크와 장치가 서로를 인증. 로밍 시 방문 네트워크 MME가 홈 HSS에 연락
3. **데이터 경로 설정**: MME가 P-GW(NAT 주소 할당), S-GW, 기지국에 두 GTP 터널 설정 → 인터넷 통신 가능

## Power Management: Sleep Modes

| 상태 | 설명 | 깊이 |
|------|------|------|
| **Discontinuous Reception** | 수백 ms 비활동 후 진입. 주기적으로(수백 ms마다) 깨어 채널 모니터링. 기지국과 연관 유지 | Light sleep |
| **Idle State** | 5-10초 비활동 후 진입. 훨씬 드물게 채널 모니터링. 셀 이동 시 기지국에 알리지 않음. 재접속 시 새 기지국과 association 필요. Paging으로 기기 위치 파악 | Deep sleep |

## 2G → 3G → 4G 아키텍처 진화

| 세대 | 코어 네트워크 | 특징 |
|------|-----------|------|
| **2G (GSM)** | 회선 교환 (음성 전용) | Base Station System, Mobile Switching Center |
| **3G (UMTS)** | 회선 교환 + 패킷 교환 병행 | 기존 음성 코어 유지 + GPRS 데이터 코어 추가 |
| **4G (LTE)** | **All-IP 패킷 교환** | 음성도 VoIP, 단일 IP 코어 |

## 출처

- [kurose-ch7](../sources/kurose-ch7.md) — Kurose 8판 7장 (7.4절)
