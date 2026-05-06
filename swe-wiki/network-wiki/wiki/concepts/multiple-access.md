---
tags: [concept, link-layer]
layer: link
related: [Ethernet, CSMA-CD, link-layer-switching]
source_count: 1
last_updated: 2026-04-08
---

# 다중 접속 프로토콜 (Multiple Access Protocols)

**브로드캐스트 링크**에서 여러 노드가 하나의 공유 채널을 통해 통신할 때, 프레임 전송을 조율하는 프로토콜. 두 개 이상의 노드가 동시에 전송하면 **충돌(Collision)**이 발생하여 모든 프레임이 손실된다.

## 이상적인 다중 접속 프로토콜의 특성

채널 전송률이 R bps일 때:

1. 1개 노드만 활성 → 처리량 **R** bps
2. M개 노드 활성 → 각 노드 평균 처리량 **R/M** bps
3. **분산형** — 단일 장애점(master node)이 없음
4. **단순** — 구현 비용이 낮음

## 분류

### 1. 채널 분할 프로토콜 (Channel Partitioning)

#### TDM (Time-Division Multiplexing)
- 시간을 **타임 프레임**으로 나누고, 각 프레임을 N개 **타임 슬롯**으로 분할
- 각 노드에 고정 타임 슬롯 할당
- 장점: 충돌 없음, 완벽한 공정성
- 단점: 1개 노드만 활성이어도 R/N으로 제한, 자기 차례까지 대기 필요

#### FDM (Frequency-Division Multiplexing)
- 채널을 N개 **주파수 대역**으로 분할, 각 노드에 전용 주파수 할당
- TDM과 동일한 장단점

#### CDMA (Code Division Multiple Access)
- 각 노드에 고유한 **코드**를 할당하여 동시 전송 가능
- 코드가 적절히 선택되면 수신자가 간섭 속에서 송신자의 데이터를 복원
- 무선 채널(셀룰러)에서 주로 사용 → Chapter 7에서 상세

### 2. 랜덤 접속 프로토콜 (Random Access)

모든 노드가 항상 **전체 채널 속도 R**로 전송하되, 충돌 시 **랜덤 지연 후 재전송**.

#### Slotted ALOHA
- 시간을 고정 크기 슬롯으로 분할, 슬롯 시작에만 전송 가능
- 충돌 시 확률 p로 재전송
- **최대 효율: 1/e ≈ 0.37** (37%만 유효 슬롯)
- 장점: 단일 활성 노드에서 전체 속도 R, 분산형, 단순
- 단점: 슬롯 동기화 필요, 빈 슬롯 + 충돌 슬롯으로 63% 낭비

#### Pure ALOHA
- 슬롯 없이, 프레임이 도착하면 **즉시 전송**
- 충돌 가능 구간이 슬롯 ALOHA의 2배 (프레임 전후 1 프레임 시간)
- **최대 효율: 1/(2e) ≈ 0.18** — slotted ALOHA의 절반

#### CSMA (Carrier Sense Multiple Access)
- **반송파 감지(Carrier Sensing)**: 전송 전에 채널이 사용 중인지 확인
- 채널이 유휴 상태면 전송, 사용 중이면 대기
- 그래도 충돌 발생 가능 — **채널 전파 지연(Propagation Delay)** 때문에 다른 노드의 전송을 즉시 감지하지 못함

#### CSMA/CD (CSMA with Collision Detection)
- **충돌 검출(Collision Detection)**: 전송 중 다른 신호가 감지되면 **즉시 전송 중단**
- CSMA보다 효율적 — 충돌 시 프레임을 끝까지 전송하지 않으므로 채널 낭비 감소

동작 절차:
1. 어댑터가 네트워크 계층에서 데이터그램을 받아 프레임 준비
2. 채널이 유휴이면 전송 시작, 사용 중이면 유휴가 될 때까지 대기
3. 전송 중 다른 어댑터의 신호 에너지를 감시
4. 충돌 감지 시 전송 중단 → 랜덤 시간 대기 → 2번으로 복귀

**이진 지수 백오프(Binary Exponential Backoff)**:
- n번째 충돌 후 K를 {0, 1, ..., 2^n − 1}에서 랜덤 선택
- 대기 시간: K × 512 bit times (이더넷 기준)
- n 최대값 = 10, 충돌이 많을수록 대기 구간이 지수적으로 증가

**CSMA/CD 효율** (근사):
```
효율 = 1 / (1 + 5·d_prop/d_trans)
```
- d_prop → 0이면 효율 → 1 (전파 지연이 작을수록 좋음)
- d_trans → ∞이면 효율 → 1 (프레임이 길수록 채널을 오래 점유)

### 3. 순번 프로토콜 (Taking-Turns)

#### 폴링(Polling)
- **마스터 노드**가 각 노드를 라운드 로빈으로 폴링
- 장점: 충돌/빈 슬롯 없음, 높은 효율
- 단점: 폴링 지연, 마스터 노드 단일 장애점
- 예: Bluetooth

#### 토큰 패싱(Token Passing)
- 마스터 없이, 특수 프레임(토큰)이 노드 간 순환
- 토큰을 받은 노드만 전송 가능
- 장점: 분산형, 효율적
- 단점: 토큰 분실/고장 시 복구 필요, 한 노드 고장이 전체에 영향
- 예: FDDI, IEEE 802.5 Token Ring

## DOCSIS — 케이블 인터넷의 다중 접속

DOCSIS(Data-Over-Cable Service Interface Specifications)는 세 가지 접속 방식을 **모두** 활용:
- **다운스트림**: FDM으로 채널 분할 (CMTS → 모뎀, 브로드캐스트)
- **업스트림**: TDM 미니슬롯 + 중앙 할당(CMTS가 MAP 메시지로 할당)
- **미니슬롯 요청**: 랜덤 접속 (충돌 시 이진 지수 백오프)

## 출처

- [kurose-ch6](../sources/kurose-ch6.md) — Kurose 8판 6장 (Section 6.3)
