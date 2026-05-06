---
tags: [concept, network, link]
layer: link
related: [lte, mobility-management, wireless-link-characteristics, nat, sdn-control-plane]
source_count: 1
last_updated: 2026-04-09
---

# 셀룰러 네트워크 아키텍처 (Cellular Network Architecture)

셀룰러 네트워크는 지리적 커버리지 영역을 **셀(cell)**로 분할하고, 각 셀에 **기지국(Base Station)**을 배치하여 **모바일 장치(Mobile Device)**에 무선 접속을 제공하는 광역 무선 네트워크이다.

## 핵심 개념

- **셀(Cell)**: 하나의 기지국이 커버하는 지리적 영역. 기지국 송신 전력, 건물 등에 따라 크기가 결정된다.
- **기지국(Base Station)**: 셀의 에지에 위치하며 무선 자원 관리, 장치 인증 조율, 터널 생성 등을 담당
- **모바일 장치(Mobile Device)**: 5계층 인터넷 프로토콜 스택을 구현하는 단말 (UE, User Equipment)
- **코어 네트워크(Core Network)**: 기지국들을 인터넷에 연결하고 인증, 이동성, 과금을 관리하는 백본

## 4G LTE 아키텍처

[[lte]] 참조. 주요 요소:

- Radio Access Network: eNode-B (기지국) ↔ Mobile Device
- All-IP EPC (Enhanced Packet Core): S-GW, P-GW, MME, HSS
- Data plane: GTP 터널 두 개 (기지국↔S-GW, S-GW↔P-GW)
- Control plane: MME 중심, HSS 연동

## 5G 아키텍처

### 5G NR (New Radio)

3GPP 표준의 5G 무선 접속 기술. 세 가지 서비스 유형:

| 서비스 | 목표 | 용도 |
|--------|------|------|
| **eMBB** (Enhanced Mobile Broadband) | 고대역폭, 중간 지연 감소 | 4K 스트리밍, AR/VR, 모바일 브로드밴드 |
| **URLLC** (Ultra Reliable Low-Latency Communications) | 1 ms 이하 지연 | 공장 자동화, 자율주행 |
| **mMTC** (Massive Machine Type Communications) | 대규모 저전력 연결 | IoT 센서, 미터링 |

### 밀리미터파 (Millimeter Wave)

5G는 두 주파수 그룹을 사용:
- **FR1** (450 MHz - 6 GHz): 4G와 유사한 대역. 초기 배포의 대부분
- **FR2** (24 - 52 GHz): **밀리미터파**. 넓은 대역폭으로 높은 전송률 가능

밀리미터파의 특성:
- **짧은 도달거리** → 더 많은 기지국 필요 (셀 밀도 증가)
- **대기/비/나뭇잎에 취약** → 실외 사용 제한
- **넓은 가용 스펙트럼** (52 - 24 = 28 GHz, 4G의 ~2 GHz 대비 14배)

### 용량 공식

```
capacity = cell density × available spectrum × spectral efficiency
           (cells/km²)    (Hz)               (bps/Hz/cell)
```

5G는 세 항 모두에서 4G를 초과 → 도심 100배 용량 증가 목표:
- 셀 밀도: 밀리미터파로 인한 소형 셀 증가
- 가용 스펙트럼: FR2의 넓은 대역
- 스펙트럴 효율: **MIMO + beam forming**으로 동일 주파수에서 10-20명 동시 서비스

### Small Cell Stations

밀리미터파의 짧은 도달거리를 보완하기 위해 기지국 사이의 커버리지 갭을 채우는 소형 기지국. 밀집 지역에서 10-100 m 간격 배치.

### 5G Core Network

| 기능 | 설명 |
|------|------|
| **UPF** (User-Plane Function) | 데이터 평면 패킷 처리를 에지로 분산 |
| **AMF** (Access and Mobility Management Function) | 4G MME의 연결/이동성 관리 부분 |
| **SMF** (Session Management Function) | 세션 관리, IP 주소 할당 (DHCP 역할) |

특징:
- 4G MME를 AMF + SMF로 **기능 분리**
- 완전한 **control/user plane 분리**
- **NFV (Network Function Virtualization)** 기반 소프트웨어 구현
- **Network slicing**: 서비스 유형별 독립 가상 네트워크

## Global Cellular Network: Network of Networks

셀룰러 네트워크도 인터넷과 마찬가지로 **네트워크의 네트워크**이다:

- 각 가입자는 **홈 네트워크(Home Network)**에 소속 (Verizon, SK Telecom 등)
- 타 캐리어 네트워크 방문 시 **방문 네트워크(Visited Network)**
- 캐리어 간 연결: 공용 인터넷 또는 **IPX**(Internet Protocol Packet eXchange) — ISP 간 IXP와 유사
- 4G 네트워크는 3G 및 2G 음성 네트워크와도 피어링 가능

## WLAN vs 셀룰러 비교

| 항목 | WiFi (WLAN) | 4G LTE |
|------|-------------|--------|
| 커버리지 | 수십 m | 수 km |
| 인증 | MAC/WPA | IMSI/HSS/MME 상호인증 |
| 이동성 관리 | 같은 서브넷 내 제한적 | 전체 네트워크 핸드오버 |
| 가입자 DB | 없음 | HSS |
| 주파수 | 비면허 (2.4/5 GHz) | 면허 대역 |

## 출처

- [[kurose-ch7]] — Kurose 8판 7장 (7.4절)
