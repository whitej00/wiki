---
tags: [concept, network]
layer: network
related: [packet-switching, delay-loss-throughput]
source_count: 1
last_updated: 2026-04-08
---

# 회선 교환 (Circuit Switching)

통신 세션 동안 송신자와 수신자 사이의 경로에 있는 자원(버퍼, 링크 전송률)을 **전용으로 예약**하는 데이터 전달 방식. 전통적인 전화망의 기반 기술이다.

## 동작 원리

1. 통신 전 종단 간 **회선(circuit)** 설정
2. 경로의 각 링크에서 일정 전송률을 전용 예약
3. 통신 동안 **보장된(guaranteed) 일정 전송률**로 데이터 전송
4. 통신 종료 시 회선 해제

## 다중화 방식

### 주파수 분할 다중화 (FDM, Frequency-Division Multiplexing)
- 링크의 주파수 대역을 여러 연결에 분할
- 각 연결이 전용 주파수 대역(bandwidth)을 사용
- 예: 전화망에서 각 연결당 4 kHz 대역

### 시분할 다중화 (TDM, Time-Division Multiplexing)
- 시간을 고정 길이 프레임으로 분할, 각 프레임을 타임 슬롯으로 분할
- 각 연결이 매 프레임에서 전용 타임 슬롯 사용
- 전송률 = 프레임 전송률 × 슬롯당 비트 수
- 예: 24슬롯 TDM, 링크 1.536 Mbps → 각 회선 64 kbps

## 한계

- **유휴 시간(silent period)** 동안 할당된 자원이 낭비됨
- 회선 설정에 시간이 소요됨 (예: 500 msec)
- 버스트 트래픽에 비효율적

## 출처

- [[kurose-ch1]] — Kurose 8판 1.3.2절
