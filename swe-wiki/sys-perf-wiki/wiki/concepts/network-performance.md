---
title: "네트워크 성능"
type: concept
tags: [network, tcp, congestion-control, bbr, socket, latency]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# 네트워크 성능

## 핵심 개념

### 레이턴시 유형

| 유형 | 설명 |
|------|------|
| Ping latency | ICMP echo RTT (localhost: 0.05ms, 10GbE: 0.2ms, SF→NY: 40ms) |
| Connection latency | TCP 3-way handshake (SYN 재전송 시 1초+ 추가) |
| TTFB (First-byte) | 연결 후 첫 바이트까지 (서버 CPU 스케줄링 포함) |

### Packet Size와 MTU
- 대부분 Ethernet MTU 1,500 bytes
- **Jumbo frames**: ~9,000 bytes (처리량/레이턴시 개선)
- TSO/GSO로 1,500 MTU의 성능 격차 일부 보완

### Connection Backlog
- **SYN backlog**: 핸드셰이크 진행 중 반-완료 연결
- **Listen backlog**: accept() 대기 중 완료 연결
- 큐 가득 차면 SYN 드롭 → 클라이언트 재전송 레이턴시

### Bufferbloat
중간 네트워크 장비의 과도한 버퍼링이 TCP 혼잡 회피를 유발. CoDel qdisc, TCP Small Queues로 대응.

## TCP 상세

### 혼잡 제어 알고리즘

| 알고리즘 | 특징 |
|----------|------|
| **CUBIC** | Linux 기본. Cubic 함수로 window 확장 |
| **BBR** | RTT+대역폭 모델링. 일부 경로에서 처리량 **3배** 개선 (Netflix) |
| **DCTCP** | ECN 기반, 데이터센터 전용 |

Linux 5.6부터 BPF로 새 혼잡 제어 알고리즘 런타임 로드 가능.

### 재전송
- **타이머 기반**: RTO 최소 200ms, exponential backoff
- **Fast retransmit**: duplicate ACK 시 즉시 재전송
- **TLP (Tail Loss Probe)**: 마지막 패킷 손실 대응

### TIME_WAIT
소켓 종료 후 ~2분 유지. 포트 고갈 문제: 초당 1,000개+ 연결 시 16비트 포트 충돌 → `tcp_tw_reuse` 설정.

### 주요 TCP 기능
- SACK, Fast Open, Timestamps, SYN Cookies
- Nagle Algorithm (TCP_NODELAY로 비활성화)
- Delayed ACKs (최대 500ms 지연)
- Initial Window 10 (Linux 기본)

### QUIC / HTTP/3
UDP 기반, Google 설계. 스트림 멀티플렉싱, 0-RTT 핸드셰이크, 페이로드 전체 암호화.

## 소프트웨어 아키텍처

### CPU Scaling
- **RSS**: NIC이 패킷을 여러 큐로 해시 → 다수 CPU 처리
- **RPS/RFS**: 소프트웨어 RSS + 소켓-CPU 매핑 최적화
- 단일 CPU만 NIC interrupt 처리 시 softirq 병목

### Kernel Bypass
- **DPDK**: 커널 스택 완전 우회, 최고 패킷 처리율. 단, 관측 도구 사용 불가.
- **XDP**: BPF 기반 빠른 경로. 커널 스택과 통합.

### Queueing Disciplines
- `fq_codel`: systemd 기본, bufferbloat 감소
- `tc(8)` 명령으로 설정

## USE Method 적용

| 메트릭 | 확인 방법 |
|--------|----------|
| Utilization | `nicstat` (%Util), `sar -n DEV 1` |
| Saturation | `ip -s link`의 overruns, TCP 재전송 |
| Errors | `ip -s link`의 errors/drops, `nstat`의 TcpRetransSegs |

## 관측 도구

| 도구 | 용도 |
|------|------|
| `ss -tiepm` | 소켓별 RTT, congestion window, 제한 요인, BBR 통계 |
| `ip -s link` | 인터페이스 통계 (바이트, 에러, 드롭) |
| `nstat` | 커널 네트워크 SNMP 통계 |
| `sar -n DEV/TCP` | 인터페이스/TCP 통계 (히스토리) |
| `nicstat` | 이용률(%Util) + 포화도(Sat) |
| `ethtool -S` | NIC 드라이버 레벨 통계 |
| `tcplife(8)` | TCP 세션 수명/처리량 (Netflix 상시 운용) |
| `tcpretrans(8)` | TCP 재전송 추적 (상태별 구분) |
| `tcpdump` | 패킷 캡처 (CPU 비용 높음, `-w` 권장) |

## 튜닝 (Netflix 프로덕션)

```
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.core.somaxconn = 1024
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_rmem = 4096 12582912 16777216
net.ipv4.tcp_wmem = 4096 12582912 16777216
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 10240 65535
```

### 소켓 옵션
- `TCP_NODELAY`: Nagle 비활성화 → 레이턴시 감소
- `SO_REUSEPORT`: 동일 포트 다중 프로세스 바인딩
- `TCP_CONGESTION`: 소켓별 혼잡 제어 알고리즘

## 관련 페이지

- [use-method](use-method.md) — USE 방법론
- [cloud-performance](cloud-performance.md) — 클라우드 네트워킹
- [bpf](../tools/bpf.md) — BPF 기반 네트워크 도구
- [benchmarking](benchmarking.md) — 네트워크 벤치마킹 (iperf, netperf)
