---
tags: [concept, network]
layer: network
related: [packet-switching, protocol-layers]
source_count: 1
last_updated: 2026-04-08
---

# 지연, 손실, 처리량 (Delay, Loss, and Throughput)

패킷 교환 네트워크의 성능을 측정하는 세 가지 핵심 지표.

## 노드 지연 (Nodal Delay)

패킷이 한 노드(라우터)를 거칠 때 겪는 총 지연:

```
d_nodal = d_proc + d_queue + d_trans + d_prop
```

### 1. 처리 지연 (Processing Delay, d_proc)
- 패킷 헤더 검사, 출력 링크 결정, 비트 오류 검사
- 고속 라우터에서 **마이크로초** 단위
- 라우터의 최대 처리량에 영향

### 2. 큐잉 지연 (Queuing Delay, d_queue)
- 출력 버퍼에서 전송 대기하는 시간
- **마이크로초 ~ 밀리초** 단위, 네트워크 혼잡도에 따라 가변적
- 4가지 지연 중 가장 복잡하고 중요

### 3. 전송 지연 (Transmission Delay, d_trans)
- 패킷의 모든 비트를 링크로 밀어내는 시간
- `d_trans = L/R` (L: 패킷 길이(비트), R: 링크 전송률(bps))
- 패킷 길이와 전송률의 함수, 라우터 간 거리와 무관

### 4. 전파 지연 (Propagation Delay, d_prop)
- 비트가 링크의 시작점에서 끝점까지 전파되는 시간
- `d_prop = d/s` (d: 링크 거리, s: 전파 속도 ≈ 2~3 × 10⁸ m/s)
- 링크 거리의 함수, 패킷 길이와 무관

## 트래픽 강도 (Traffic Intensity)

```
Traffic Intensity = La/R
```
- a: 패킷 도착률 (packets/sec), L: 패킷 길이 (bits), R: 전송률 (bps)
- La/R → 1 에 가까워지면 평균 큐잉 지연이 **급격히 증가**
- La/R > 1 이면 큐가 무한히 증가 → **시스템 설계 시 트래픽 강도가 1을 넘지 않도록 해야 함**

## 패킷 손실 (Packet Loss)

- 출력 버퍼의 용량은 유한함
- 버퍼가 가득 차면 도착 패킷 또는 대기 중 패킷이 **드롭됨**
- 트래픽 강도가 증가할수록 손실 비율 증가
- 손실된 패킷은 종단 간 재전송으로 복구 가능 (예: TCP)

## 종단 간 지연 (End-to-End Delay)

N-1개의 라우터를 거치는 경로에서 (혼잡 없는 경우):

```
d_end-end = N × (d_proc + d_trans + d_prop)
```

### Traceroute
- 경로 상의 각 라우터까지의 RTT를 측정하는 도구
- 각 라우터에 3개의 특수 패킷을 전송하여 왕복 시간 기록
- RFC 1393에 상세 설명

## 처리량 (Throughput)

수신 호스트가 데이터를 받는 속도 (bits/sec).

- **순간 처리량**: 특정 시점의 수신 속도
- **평균 처리량**: F비트 전송에 T초 소요 시 F/T bps

### 병목 링크 (Bottleneck Link)
경로 상에서 전송률이 가장 낮은 링크가 처리량을 결정:

```
Throughput = min{R_1, R_2, ..., R_N}
```

오늘날 인터넷에서 병목은 보통 **액세스 네트워크**(서버 또는 클라이언트 측)에 존재한다. 단, 코어 링크에 여러 트래픽 흐름이 공유되면 코어가 병목이 될 수도 있다.

## 출처

- [kurose-ch1](../sources/kurose-ch1.md) — Kurose 8판 1.4절
