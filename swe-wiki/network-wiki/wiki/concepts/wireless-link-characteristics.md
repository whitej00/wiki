---
tags: [concept, link]
layer: link
related: [wifi-80211, cdma, multiple-access, error-detection-correction, lte]
source_count: 1
last_updated: 2026-04-09
---

# 무선 링크 특성 (Wireless Link Characteristics)

무선 링크는 유선 링크와 비교하여 **신호 감쇠, 간섭, 다중경로 전파** 등의 고유한 문제를 가지며, 이로 인해 높은 비트 오류율(BER)과 특수한 매체 접근 문제가 발생한다.

## 유선 대비 무선의 차이

### 1. 신호 감쇠 (Decreasing Signal Strength / Path Loss)

전자기 복사는 물질을 통과하면서 감쇠한다. 자유 공간에서도 거리가 증가하면 신호가 분산되어 강도가 감소한다. 이를 **경로 손실(Path Loss)**이라 한다.

### 2. 타 소스 간섭 (Interference from Other Sources)

같은 주파수 대역을 사용하는 다른 무선 장치와 간섭이 발생한다. 예를 들어, 2.4 GHz 대역에서 802.11b 무선 LAN과 무선 전화기가 동시에 동작하면 양쪽 모두 성능이 저하된다. 전자레인지, 모터 등 환경 전자기 잡음도 간섭 원인이 된다. 이 때문에 최신 802.11 표준들은 5 GHz 대역도 지원한다.

### 3. 다중경로 전파 (Multipath Propagation)

전자기파가 물체와 지면에 반사되어 서로 다른 경로로 수신기에 도달한다. 수신 신호가 **뭉개지는(blurring)** 현상이 발생하며, 송수신기 사이 물체 이동에 따라 시간에 따라 변한다.

## SNR과 BER

**신호 대 잡음비(Signal-to-Noise Ratio, SNR)**는 수신 신호 강도와 배경 잡음의 상대적 크기로, dB 단위로 측정한다 (20 × log₁₀(신호 진폭 / 잡음 진폭)).

**비트 오류율(Bit Error Rate, BER)**은 전송된 비트가 수신 측에서 오류로 수신될 확률이다.

핵심 관계:

- **같은 변조 기법에서 SNR이 높을수록 BER은 낮다** — 송신 전력을 높이면 SNR이 올라가지만, 에너지 소비 증가 및 타 송신자 간섭 증가라는 부작용이 있다.
- **같은 SNR에서 전송률이 높은 변조 기법은 BER이 높다** — 예: SNR 10 dB일 때 BPSK(1 Mbps)는 BER < 10⁻⁷이지만 QAM16(4 Mbps)은 BER ≈ 10⁻¹로 실용적이지 못하다.
- **물리 계층 변조 기법을 채널 상태에 맞게 동적으로 선택할 수 있다** — adaptive modulation and coding, [[wifi-80211]]과 [[lte]] 모두 활용.

## Hidden Terminal 문제

무선 채널에서 A가 B에게 전송 중일 때, C가 A의 전송을 감지하지 못하고(물리적 장애물 또는 거리) B에게 프레임을 보내면 B에서 충돌이 발생한다. 이를 **hidden terminal problem**이라 한다.

유선의 CSMA/CD와 달리 무선에서는 충돌 **검출(detection)**이 불가능하다:
- 송신 신호 대비 수신 신호가 너무 약해 동시 감지 어려움
- hidden terminal과 fading으로 인해 모든 충돌을 감지할 수 없음

이 문제를 완화하기 위해 802.11은 **[[wifi-80211|RTS/CTS]]** 메커니즘을 사용한다.

## Fading

전파 신호 강도가 위치에 따라 급격히 변하는 현상이다. A와 C가 각각 B에게 도달할 수 없지만 B에서의 신호가 서로 간섭하는 상황이 발생할 수 있다. Hidden terminal 문제와 함께 무선 네트워크의 다중 접속을 유선보다 훨씬 복잡하게 만든다.

## 무선 링크 프로토콜의 대응

| 문제 | 대응 |
|------|------|
| 높은 BER | CRC + link-layer ARQ (ACK/재전송), FEC |
| Hidden terminal | RTS/CTS 채널 예약 |
| 가변 채널 상태 | Adaptive modulation / rate adaptation |
| 에너지 제약 | Power management (sleep/wake) |

## 출처

- [[kurose-ch7]] — Kurose 8판 7장 (7.2절)
