---
tags: [concept, transport-layer]
layer: transport
related: [TCP, UDP, QUIC, flow-control, reliable-data-transfer]
source_count: 1
last_updated: 2026-04-08
---

# 혼잡 제어 (Congestion Control)

네트워크 혼잡 — 너무 많은 소스가 너무 빠르게 데이터를 전송하여 네트워크가 처리할 수 없는 상태 — 을 감지하고, 송신자의 전송률을 조절하여 혼잡을 방지하거나 완화하는 메커니즘이다. [흐름 제어](flow-control.md)가 수신자 보호인 반면, 혼잡 제어는 **네트워크 전체의 이익**을 위한 것이다.

## 혼잡의 원인과 비용

### 시나리오 1: 무한 버퍼 라우터, 2개 송신자

- 처리량은 링크 용량의 절반(R/2)에 수렴
- 전송률이 R/2에 근접할수록 **큐잉 지연이 무한히 증가**
- 비용: 큰 큐잉 지연

### 시나리오 2: 유한 버퍼 라우터, 재전송

- 패킷 손실(버퍼 오버플로)로 재전송 필요 → **offered load**(λ'_in)가 원본 데이터율(λ_in)보다 큼
- 이상적 재전송(확실히 손실된 경우만): 처리량 = R/3 (offered load = R/2일 때)
- 조기 타임아웃으로 불필요한 재전송 시: 처리량 = R/4까지 감소
- 비용: **(1) 손실 패킷 재전송, (2) 불필요한 중복 재전송으로 대역폭 낭비**

### 시나리오 3: 다중 홉, 유한 버퍼

- 4개 송신자, 겹치는 2-hop 경로
- offered load가 극도로 커지면, 한 연결의 **처리량이 0**으로 수렴 가능
- 비용: **드롭된 패킷이 이미 소비한 업스트림 전송 용량이 완전히 낭비** (congestion collapse)

## 혼잡 제어 접근 방식

### End-to-End 혼잡 제어

네트워크 계층이 혼잡 정보를 **명시적으로 제공하지 않음**. 종단 시스템이 관찰된 네트워크 동작(패킷 손실, 지연)으로 혼잡을 **추론**한다.

[TCP](../protocols/tcp.md)가 이 접근을 사용: 세그먼트 손실(타임아웃 또는 3중복 ACK)을 혼잡의 신호로 해석하고 윈도우 크기를 줄인다.

### Network-Assisted 혼잡 제어

라우터가 혼잡 상태에 대한 **명시적 피드백**을 송신자/수신자에게 제공한다.

피드백 방식:
1. **직접 피드백**: 라우터가 송신자에게 choke 패킷을 직접 전송
2. **간접 피드백**: 라우터가 송신자→수신자 패킷에 혼잡 표시 → 수신자가 송신자에게 알림 (1 RTT 소요)

예: ATM ABR, **ECN(Explicit Congestion Notification)**

## TCP 혼잡 제어

### 혼잡 윈도우 (cwnd)

TCP 송신자는 **혼잡 윈도우(cwnd)** 변수를 유지하여 전송률을 제한한다:

```
LastByteSent − LastByteAcked ≤ min{cwnd, rwnd}
```

전송률 ≈ cwnd / RTT bytes/sec

### 기본 원리

1. **손실 = 혼잡 신호**: 세그먼트 손실 시 전송률 감소
2. **ACK = 네트워크 정상 신호**: ACK 도착 시 전송률 증가 가능
3. **대역폭 탐색(Bandwidth Probing)**: 손실이 발생할 때까지 전송률을 증가시키고, 손실 시 감소 후 다시 증가

TCP는 ACK를 트리거(클럭)로 사용하여 cwnd를 증가시키므로 **self-clocking**이라 한다.

### TCP 혼잡 제어 알고리즘 (RFC 5681)

세 가지 주요 구성 요소:

#### 1. Slow Start (느린 시작)

연결 시작 또는 타임아웃 후:
- cwnd = **1 MSS** (초기 전송률 ≈ MSS/RTT)
- **매 ACK마다** cwnd += 1 MSS → RTT마다 cwnd **2배** (지수적 증가)
- 이름과 달리 **빠르게** 증가하나, 아주 작은 값에서 시작하므로 "slow start"

종료 조건:
1. **타임아웃**: cwnd = 1 MSS, ssthresh = cwnd/2로 설정. slow start 재시작
2. **cwnd ≥ ssthresh**: congestion avoidance로 전환
3. **3중복 ACK**: ssthresh = cwnd/2, cwnd = ssthresh + 3·MSS. fast recovery 진입

#### 2. Congestion Avoidance (혼잡 회피)

cwnd가 ssthresh에 도달한 후:
- **RTT마다 cwnd를 1 MSS씩** 증가 (선형 증가)
- 구현: 매 ACK마다 cwnd += MSS · (MSS/cwnd) 바이트

종료 조건:
1. **타임아웃**: cwnd = 1 MSS, ssthresh = cwnd/2. slow start 재시작
2. **3중복 ACK**: ssthresh = cwnd/2, cwnd = ssthresh + 3·MSS. fast recovery 진입

#### 3. Fast Recovery (빠른 복구)

3중복 ACK(타임아웃보다 덜 심각한 손실)에 대한 대응:
- **매 중복 ACK마다** cwnd += 1 MSS (네트워크가 일부 세그먼트를 전달 중이라는 신호)
- 손실 세그먼트에 대한 **새 ACK 도착** 시: cwnd = ssthresh, congestion avoidance 진입

fast recovery는 권장사항이며 필수는 아님 (RFC 5681).

### TCP Tahoe vs TCP Reno

| | TCP Tahoe | TCP Reno |
|---|-----------|----------|
| 타임아웃 시 | cwnd = 1 MSS | cwnd = 1 MSS |
| 3중복 ACK 시 | cwnd = 1 MSS (slow start) | cwnd = cwnd/2 (fast recovery) |
| fast recovery | 없음 | 있음 |

TCP Reno가 3중복 ACK에 덜 공격적으로 대응하여 더 높은 처리량을 달성한다.

### AIMD (Additive Increase, Multiplicative Decrease)

slow start 이후 TCP의 혼잡 제어 동작을 요약하면:
- **Additive Increase**: 혼잡 없을 때 RTT마다 cwnd를 1 MSS씩 증가
- **Multiplicative Decrease**: 손실(3중복 ACK) 시 cwnd를 절반으로 감소

이 패턴은 **톱니파형(Sawtooth)** 그래프를 생성한다.

TCP Reno의 평균 처리량:
```
평균 처리량 = 0.75 · W / RTT    (W = 손실 시점의 윈도우 크기)
```

### TCP CUBIC

TCP Reno의 AIMD보다 효율적인 대역폭 탐색을 위한 알고리즘. RFC 8312. **Linux 기본 TCP**.

핵심 아이디어: 손실 직전의 전송률로 빠르게 복귀하고, 그 근처에서는 조심스럽게 탐색.

- W_max: 마지막 손실 감지 시의 cwnd 크기
- K: cwnd가 다시 W_max에 도달할 미래 시점
- cwnd는 **큐빅 함수(cube function)**로 증가:
  - t가 K에서 멀면: 큰 폭으로 증가 (빠르게 W_max 근처로 복귀)
  - t가 K 근처면: 작은 폭으로 증가 (조심스러운 탐색)
  - t > K이면: 다시 큰 폭으로 증가 (새 운영점 탐색)

웹 서버 측정에서 TCP CUBIC이 TCP Reno보다 널리 사용되는 것으로 확인 (~50% 이상).

### Explicit Congestion Notification (ECN)

RFC 3168. **Network-assisted** 혼잡 제어의 인터넷 구현.

동작:
1. **라우터**: IP 데이터그램 헤더의 ToS 필드 내 ECN 비트(2비트)를 설정하여 혼잡 표시
2. **수신 TCP**: ECN 표시를 감지하면, ACK 세그먼트의 **ECE(ECN Echo)** 비트를 설정하여 송신자에게 알림
3. **송신 TCP**: ECE 수신 시 cwnd를 절반으로 줄이고, 다음 세그먼트에 **CWR(Congestion Window Reduced)** 비트 설정

장점: 패킷 손실 **전에** 혼잡을 감지하여 선제적 대응 가능

관련 프로토콜: DCCP(RFC 4340), DCTCP(RFC 8257), DCQCN 등도 ECN 활용.

### 지연 기반 혼잡 제어 (Delay-based)

패킷 손실이 아닌 **RTT 증가**를 혼잡 신호로 사용하는 접근.

#### TCP Vegas

- RTT_min: 관측된 최소 RTT (비혼잡 경로의 RTT)
- 비혼잡 처리량: cwnd / RTT_min
- 실측 처리량이 비혼잡 처리량에 가까우면 → cwnd 증가 가능
- 실측 처리량이 비혼잡 처리량보다 현저히 낮으면 → 혼잡, cwnd 감소
- 철학: **"Keep the pipe just full, but no fuller"**

#### BBR (Bottleneck Bandwidth and Round-trip propagation time)

Google이 개발한 지연 기반 혼잡 제어 프로토콜. TCP Vegas의 아이디어를 확장.
- Google B4 네트워크(데이터센터 연결), YouTube, Google 웹 서버에 배포
- CUBIC을 대체하여 사용 중

기타 지연 기반: TIMELY(데이터센터), Compound TCP/CTPC, FAST(고속 장거리)

## 공정성 (Fairness)

### AIMD의 공정성

K개의 TCP 연결이 용량 R의 병목 링크를 공유할 때, 혼잡 제어가 **공정(fair)**하다면 각 연결의 평균 전송률 ≈ R/K.

AIMD는 공정성으로 수렴한다:
- 두 연결이 동일 RTT, 동일 MSS를 가지면
- 총 처리량 < R일 때: 둘 다 1 MSS/RTT씩 증가 (45도 방향)
- 총 처리량 > R일 때: 둘 다 절반으로 감소
- 이 과정이 반복되면 **균등 대역폭 분배선**으로 수렴

### 공정성의 한계

1. **RTT 차이**: RTT가 작은 연결이 cwnd를 더 빠르게 키워 더 큰 대역폭 차지
2. **UDP 트래픽**: UDP는 혼잡 제어가 없어 TCP 트래픽을 밀어낼 수 있음
3. **병렬 TCP 연결**: 하나의 애플리케이션이 N개의 TCP 연결을 열면, 대역폭의 N/(N+K-N) 이상을 차지 가능. 웹 브라우저에서 흔히 사용

## TCP Splitting

클라우드 서비스(검색, 이메일 등)의 성능 최적화 기법.

문제: 원격 데이터센터까지 TCP 연결 시 slow start로 인한 지연 ≈ 4·RTT + 처리 시간

해결: 클라이언트 근처에 **프론트엔드 서버** 배치
- 클라이언트 ↔ 프론트엔드: 짧은 TCP 연결
- 프론트엔드 ↔ 데이터센터: 큰 cwnd의 지속 TCP 연결
- 응답 시간 ≈ 4·RTT_FE + RTT_BE + 처리시간 (RTT_FE ≈ 0이면 ≈ RTT)

Google, Akamai 등이 CDN 서버에서 TCP splitting을 활용.

## 출처
- [kurose-ch3](../sources/kurose-ch3.md)
