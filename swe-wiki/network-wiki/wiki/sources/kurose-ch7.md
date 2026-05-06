---
tags: [source, kurose, wireless, mobility]
source: "Kurose & Ross, Computer Networking: A Top-Down Approach, 8th Edition, Chapter 7"
pages: "561-628"
topics: [wireless-link-characteristics, WiFi, 802.11, CDMA, Bluetooth, 4G-LTE, 5G, mobility-management, Mobile-IP, handover]
last_updated: 2026-04-09
---

# Kurose 8판 7장: Wireless and Mobile Networks

## 핵심 내용 요약

무선(wireless)과 이동성(mobility)을 **별개의 문제**로 구분하여 다룬다. 무선 링크의 특성(높은 BER, hidden terminal, fading)은 링크 계층에서, 이동성(handover, roaming)은 네트워크 계층에서 각각 해결해야 하는 과제이다.

### 7.1 Introduction
- 무선 네트워크 구성요소: wireless host, wireless link, base station, network infrastructure
- 분류 기준: (i) single-hop vs multi-hop, (ii) infrastructure vs infrastructure-less
- Infrastructure mode vs ad hoc network, handoff/handover 개념 도입

### 7.2 Wireless Links and Network Characteristics
- 유선 대비 무선 링크의 차이: 경로 손실(path loss), 간섭(interference), 다중경로 전파(multipath propagation)
- **SNR**(Signal-to-Noise Ratio)과 **BER**(Bit Error Rate)의 관계
- 변조 기법(BPSK, QAM16, QAM256)별 전송률 vs BER 트레이드오프
- **Hidden terminal problem**과 **fading** 문제
- **CDMA**(Code Division Multiple Access): chipping rate, 직교 코드, 다중 송신자 간 간섭 극복

### 7.3 WiFi: 802.11 Wireless LANs
- **802.11 아키텍처**: BSS(Basic Service Set), AP(Access Point), SSID, 채널 할당 (1, 6, 11)
- **Passive/active scanning**, association 과정, 인증 (MAC 기반, WPA)
- **CSMA/CA**: collision avoidance 방식, DIFS/SIFS, binary exponential backoff
  - 802.11은 collision detection 불가 → 프레임 전체 전송 후 ACK 확인
  - **Link-layer ACK**: CRC 통과 시 SIFS 후 ACK 반환
- **RTS/CTS**: hidden terminal 문제 완화, 채널 예약 메커니즘
- **802.11 프레임**: 4개 주소 필드(수신 MAC, 송신 MAC, 라우터 인터페이스 MAC, ad hoc용), duration, sequence number, frame control
- **Same-subnet mobility**: 같은 IP 서브넷 내 BSS 이동 시 IP/TCP 유지, 스위치 포워딩 테이블 갱신
- **Rate adaptation**: SNR 변화에 따라 변조 기법 동적 선택 (TCP congestion control과 유사한 "probing" 철학)
- **Power management**: sleep/wake, beacon frame 기반, 99% 시간 절전 가능
- **Bluetooth**: 2.4 GHz ISM, FHSS, piconet (최대 8 active + 255 parked), neighbor discovery, paging

### 7.4 Cellular Networks: 4G and 5G
- **4G LTE 아키텍처**: mobile device(UE), base station(eNode-B), MME, HSS, S-GW, P-GW
  - **Data plane**: 기지국-S-GW, S-GW-P-GW 두 GTP 터널로 구성
  - **Control plane**: MME 중심 (인증, 위치 추적, 경로 설정)
  - IMSI: 64비트 글로벌 가입자 식별자 (SIM 카드)
- **LTE 프로토콜 스택**: Packet Data Convergence(헤더 압축/암호화), Radio Link Control(단편화/ARQ), MAC(스케줄링/FEC)
- **LTE Radio Access**: OFDM, 0.5ms 타임슬롯, 주파수 분할 + 시분할 결합
- **Network attachment**: 기지국 탐색 → 상호 인증(MME-HSS) → 데이터 경로 설정
- **Power management**: discontinuous reception (light sleep) + idle state (deep sleep), paging
- **5G NR**: eMBB(고대역폭), URLLC(초저지연 1ms), mMTC(대규모 IoT)
- **밀리미터파**(FR2, 24-52 GHz): 짧은 도달거리, 대기 간섭 취약, MIMO + beam forming으로 보완
- **5G Core**: NFV 기반, UPF/AMF/SMF로 기능 분리, network slicing

### 7.5 Mobility Management: Principles
- 이동성 스펙트럼: (a) 이동 중 전원 끔 → (b) 같은 AP 내 이동 → (c) 단일 제공자 내 이동 → (d) 제공자 간 이동
- **Home network / Visited network** 개념, HSS의 역할
- **Indirect routing**: correspondent → home gateway → visited gateway → mobile device (triangle routing 문제)
- **Direct routing**: correspondent가 HSS 조회 → visited network로 직접 터널링 (더 효율적이나 복잡)
- 터널링(encapsulation/decapsulation), local breakout 개념

### 7.6 Mobility Management in Practice
- **4G/5G mobility 4단계**: (1) base station association (2) control-plane configuration (MME-HSS) (3) data-plane tunnel setup (4) handover
- **Handover 7단계 절차**: source BS → target BS Handover Request → resource pre-allocation → mobile switch → forwarding → MME/S-GW 터널 재구성 → 자원 해제
- **Mobile IP** (RFC 5944): home agent, foreign agent, care-of-address, 간접 라우팅 — 4G/5G와 유사한 아키텍처

### 7.7 Wireless and Mobility: Impact on Higher-Layer Protocols
- TCP의 혼잡 제어가 무선 링크의 비트 오류/핸드오버 손실을 혼잡으로 오인 → 불필요한 전송률 감소
- 해결 접근: (1) local recovery (ARQ/FEC) (2) TCP sender awareness (3) split-connection
- 애플리케이션 계층: 대역폭 제약, 위치 기반 서비스의 가능성

## 다룬 토픽 목록

| 토픽 | 위키 페이지 |
|------|-----------|
| 무선 링크 특성 | [[wireless-link-characteristics]] |
| CDMA | [[cdma]] |
| WiFi 802.11 | [[wifi-80211]] |
| Bluetooth | [[bluetooth]] |
| 셀룰러 네트워크 아키텍처 | [[cellular-network-architecture]] |
| 4G LTE | [[lte]] |
| 이동성 관리 | [[mobility-management]] |

## 위키 반영 페이지

- 생성: [[wifi-80211]], [[bluetooth]], [[lte]], [[wireless-link-characteristics]], [[cdma]], [[mobility-management]], [[cellular-network-architecture]]
- 업데이트: [[layers/link]], [[index]]
