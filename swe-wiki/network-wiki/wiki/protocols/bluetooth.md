---
tags: [protocol, link]
layer: link
standard: IEEE 802.15.1
related: [wifi-80211, wireless-link-characteristics, cdma, multiple-access]
source_count: 1
last_updated: 2026-04-09
---

# Bluetooth

**Bluetooth**는 짧은 거리(수십 미터 이하)에서 저전력으로 동작하는 **무선 개인 영역 네트워크(Wireless Personal Area Network, WPAN)** 기술이다. 키보드, 마우스, 이어폰, 스마트워치 등 주변 장치 연결에 널리 사용된다.

## 한 줄 요약

2.4 GHz ISM 대역에서 FHSS(주파수 호핑 확산 스펙트럼)를 사용하는 ad hoc 피코넷 기반 단거리 무선 프로토콜.

## 네트워크 구조: 피코넷 (Piconet)

Bluetooth는 **인프라(AP) 없이** ad hoc으로 동작한다. 장치들은 스스로 **피코넷(piconet)**을 구성한다:

- **중앙 컨트롤러(Centralized Controller)** 1대 — 피코넷의 마스터 역할
  - 클록 결정 (TDM 슬롯 경계)
  - 주파수 호핑 시퀀스 결정
  - 클라이언트 진입 제어
  - 송신 전력 제어 (100 mW / 2.5 mW / 1 mW)
  - polling 방식으로 클라이언트에게 전송 허가
- **클라이언트(Client)** 최대 **7대** (active)
- **파킹(Parked) 장치** 최대 **255대** — sleep 모드, 중앙 컨트롤러가 활성화할 때까지 통신 불가

## 물리 계층

- **주파수**: 2.4 GHz ISM (비면허) 대역
- **FHSS (Frequency-Hopping Spread Spectrum)**: 79개 채널 중 하나에서 625 μs 타임슬롯마다 의사랜덤으로 주파수 변경
  - ISM 대역의 간섭(전자레인지, 차고 도어 등)에 강건
  - 간섭 시 슬롯 일부만 영향 받음
- **데이터 속도**: 최대 3 Mbps

## Self-Organizing: 네트워크 구성

### 1. 이웃 발견 (Neighbor Discovery)

중앙 컨트롤러가 32개 **inquiry 메시지**를 서로 다른 주파수로 브로드캐스트하며, 최대 128회 반복한다. 클라이언트는 선택한 주파수에서 inquiry를 수신하면 0~0.3초 랜덤 백오프 후 장치 ID를 포함한 응답을 보낸다 (이더넷의 binary backoff와 유사).

### 2. 페이징 (Bluetooth Paging)

발견된 클라이언트를 피코넷에 초대하는 과정:
1. 중앙 컨트롤러가 32개 동일한 paging 초대 메시지를 서로 다른 주파수로 전송
2. 클라이언트가 ACK 응답
3. 중앙 컨트롤러가 주파수 호핑 정보, 클록 동기, 멤버 주소를 전송
4. 클라이언트가 주파수 호핑 패턴으로 통신 시작

802.11의 AP association과 유사한 과정이다.

## 사용하는 링크 계층 기법

- TDM (시분할 다중화)
- 주파수 분할 (FHSS)
- 랜덤 백오프 (neighbor discovery)
- Polling (전송 허가)
- 오류 검출/정정
- ACK/NAK 기반 ARQ

## 출처

- [kurose-ch7](../sources/kurose-ch7.md) — Kurose 8판 7장 (7.3.6절)
