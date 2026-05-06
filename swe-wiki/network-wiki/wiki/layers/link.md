---
tags: [layer, link]
layer: link
related: [network, Ethernet, ARP, mac-address, error-detection-correction, multiple-access, link-layer-switching, vlan, mpls, wifi-80211, bluetooth, wireless-link-characteristics, cdma]
source_count: 3
last_updated: 2026-04-09
---

# 링크 계층 (Link Layer)

네트워크 계층과 물리 계층 사이에 위치하며, **인접한 두 노드 사이의 개별 링크(Individual Link)**를 통해 데이터그램을 이동시키는 계층이다.

## 역할

네트워크 계층(IP)이 **종단 간(end-to-end)** 경로를 통해 데이터그램을 전달하는 반면, 링크 계층은 경로 상의 **개별 링크(hop)** 하나를 담당한다. 각 링크에서 네트워크 계층 데이터그램은 **링크 계층 프레임(Link-Layer Frame)**으로 캡슐화되어 전송된다.

## 용어

- **노드(Node)** — 링크 계층 프로토콜을 실행하는 모든 장치 (호스트, 라우터, 스위치, WiFi AP 등)
- **링크(Link)** — 경로 상의 인접한 노드를 연결하는 통신 채널

## 링크 계층이 제공하는 서비스

| 서비스 | 설명 |
|--------|------|
| **프레이밍(Framing)** | 네트워크 계층 데이터그램을 프레임에 캡슐화 (데이터 필드 + 헤더 필드) |
| **링크 접근(Link Access)** | [매체 접근 제어(MAC)](../concepts/multiple-access.md) 프로토콜로 프레임 전송 규칙 조율 |
| **신뢰적 전달(Reliable Delivery)** | ACK/재전송으로 오류 없는 전달 보장 (주로 무선 링크에서 사용, 유선은 불필요) |
| **[오류 검출/정정](../concepts/error-detection-correction.md)** | 비트 오류를 검출하고, 경우에 따라 정정 (패리티, CRC 등) |

## 링크 계층의 구현 위치

링크 계층은 **하드웨어와 소프트웨어의 결합**으로 구현된다:

- **하드웨어**: 대부분의 기능은 **네트워크 어댑터(Network Adapter)** 또는 **NIC(Network Interface Controller)** 칩에서 구현 — 프레이밍, 링크 접근, 오류 검출 등
- **소프트웨어**: 링크 계층 주소 조립, 컨트롤러 활성화 등 상위 기능은 호스트 CPU에서 실행

링크 계층은 프로토콜 스택에서 **소프트웨어와 하드웨어가 만나는 지점**이다.

## 링크의 두 가지 유형

1. **점대점 링크(Point-to-Point Link)** — 한 송신자와 한 수신자 (예: 이더넷 스위치-호스트 연결, PPP)
2. **브로드캐스트 링크(Broadcast Link)** — 여러 노드가 공유 채널에 연결 (예: 무선 LAN, 과거 이더넷 버스, HFC)

브로드캐스트 링크에서는 [다중 접속 문제](../concepts/multiple-access.md)가 발생하며, 이를 해결하기 위한 MAC 프로토콜이 필요하다.

## 주요 프로토콜 및 개념

### 유선
- [Ethernet](../protocols/ethernet.md) — 가장 널리 사용되는 유선 LAN 기술
- [ARP](../protocols/arp.md) — IP 주소를 [MAC 주소](../concepts/mac-address.md)로 변환
- [link-layer-switching](../concepts/link-layer-switching.md) — 스위치의 self-learning과 필터링/포워딩
- [vlan](../concepts/vlan.md) — 가상 LAN으로 트래픽 격리
- [mpls](../concepts/mpls.md) — 라벨 기반 스위칭으로 링크 가상화
- [data-center-networking](../concepts/data-center-networking.md) — 데이터 센터의 계층적 네트워크 구조

### 무선
- [wifi-80211](../protocols/wifi-80211.md) — IEEE 802.11 무선 LAN (WiFi). CSMA/CA, link-layer ACK, RTS/CTS
- [bluetooth](../protocols/bluetooth.md) — Bluetooth (WPAN). 2.4 GHz FHSS, piconet
- [lte](../protocols/lte.md) — 4G LTE 셀룰러 네트워크. OFDM, GTP 터널링
- [wireless-link-characteristics](../concepts/wireless-link-characteristics.md) — 무선 링크 특성 (경로 손실, 간섭, 다중경로, SNR/BER)
- [cdma](../concepts/cdma.md) — 코드 분할 다중 접속 (CDMA)

## 무선 보안

- **WEP → WPA1 → WPA2 → WPA3**: 802.11 보안 프로토콜의 진화. WEP은 심각한 결함 발견 후 WPA2(AES 기반)가 표준. WPA3(2018)은 nonce 재사용 공격 대응
- **WPA2 4-way handshake**: 공유 비밀 K_AS-M + nonce 교환으로 상호 인증 및 세션 키 K_M-AP 유도
- **4G AKA (Authentication and Key Agreement)**: IMSI + K_HSS-M으로 상호 인증 후 세션 키 유도
- **5G 변경사항**: 홈 네트워크가 인증 결정권, IMSI 공개키 암호화 전송, AKA' 프로토콜 추가
- 프로토콜 스택: EAP-TLS / EAP / EAPoL(무선 구간) + RADIUS(유선 구간)

> 자세한 내용: [kurose-ch8](../sources/kurose-ch8.md) 8.8절

## 출처

- [kurose-ch6](../sources/kurose-ch6.md) — Kurose 8판 6장
- [kurose-ch7](../sources/kurose-ch7.md) — Kurose 8판 7장 (무선 및 이동 네트워크)
- [kurose-ch8](../sources/kurose-ch8.md) — Kurose 8판 8.8절 (무선/셀룰러 보안)
