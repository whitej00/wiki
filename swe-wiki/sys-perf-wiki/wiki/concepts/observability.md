---
title: "관측 가능성 (Observability)"
type: concept
tags: [observability, tracing, profiling, monitoring, counters]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# 관측 가능성 (Observability)

## 왜 관측이 중요한가

관측 가능성이란 관찰을 통해 시스템을 이해하는 것이다. 프로덕션 환경에서는 **실험(벤치마크) 도구보다 관측 도구를 먼저** 사용해야 한다. 실험 도구는 리소스 경합으로 워크로드를 방해할 수 있다.

## 도구 유형

### 1. Fixed Counters (고정 카운터)

커널이 유지하는 정수 변수. 성능 도구가 이를 읽어 통계(변화율, 평균)를 계산한다.

```
카운터 → 통계 → 메트릭 → 알림
```

도구 예: `vmstat`, `iostat`, `mpstat`, `sar`, `netstat`

### 2. Profiling (프로파일링)

일정 시간 간격으로 시스템 상태를 **샘플링**하여 대상의 그림을 그린다.

- CPU 프로파일링: 99Hz로 on-CPU 코드 경로 샘플 수집
- **Flame Graph**로 시각화: 너비 = CPU 시간, 세로 = 코드 경로
- 도구: [[perf]], `profile(8)` (BCC)

### 3. Tracing (트레이싱)

이벤트 기반 기록. 이벤트 데이터를 캡처하여 분석하거나 실시간 요약 생성.

#### 정적 인스트루멘테이션 (Static Instrumentation)
- 소스 코드에 하드코딩된 계측 포인트
- **Tracepoints**: 커널 정적 계측 (안정적 API)
- **USDT**: 유저 공간 정적 계측

#### 동적 인스트루멘테이션 (Dynamic Instrumentation)
- 실행 중 소프트웨어의 인-메모리 명령어를 수정하여 계측
- **kprobes**: 커널 함수 (2004)
- **uprobes**: 유저 함수
- "촛불(카운터)이 아닌 손전등(동적 트레이싱)" — 어디든 비출 수 있다

#### BPF 트레이싱
- 커널 내에서 필터링/집계 수행 → 프로덕션 사용 가능한 낮은 오버헤드
- 도구: [[bpf]] (BCC, bpftrace)

### 4. Monitoring (모니터링)

시간 경과에 따라 성능 통계를 기록. Grafana 등으로 시각화. 경보 시스템으로 문제 통보.

## Observability Sources

### /proc 파일시스템

프로세스 및 커널 통계를 파일 인터페이스로 제공.

| 파일 | 내용 |
|------|------|
| `/proc/cpuinfo` | CPU 정보 |
| `/proc/meminfo` | 메모리 통계 |
| `/proc/diskstats` | 디스크 I/O 통계 |
| `/proc/net/dev` | 네트워크 인터페이스 통계 |
| `/proc/PID/status` | 프로세스 상태 |
| `/proc/PID/maps` | 메모리 매핑 |
| `/proc/pressure/*` | PSI (Pressure Stall Information) |

### /sys 파일시스템

장치 및 커널 모듈 통계/설정.

### Hardware Counters (PMCs)

프로세서에 내장된 레지스터로 저수준 CPU 활동 측정.
- CPU 사이클, 명령어 수, L1/L2/L3 캐시 히트/미스
- IPC(Instructions Per Cycle) 계산의 핵심
- 도구: `perf stat`, `turbostat`

## Crisis Tools (위기 도구)

장애 발생 시 즉시 사용할 도구 목록:

```bash
uptime          # 부하 평균
dmesg -T        # 커널 메시지
vmstat 1        # 전체 시스템 개요
mpstat -P ALL 1 # CPU별 통계
pidstat 1       # 프로세스별 CPU
iostat -xz 1    # 디스크 I/O
free -m         # 메모리 사용
sar -n DEV 1    # 네트워크 I/O
sar -n TCP 1    # TCP 통계
top             # 종합 모니터링
```

이것이 Brendan Gregg의 **"Linux Perf Analysis in 60 Seconds"** 체크리스트다.

## sar (System Activity Reporter)

가장 오래된 범용 관측 도구. 과거 데이터 보관 및 실시간 출력 모두 지원.

| 옵션 | 내용 |
|------|------|
| `-u` | CPU 활용률 |
| `-r` | 메모리 활용률 |
| `-B` | 페이징 통계 |
| `-d` | 디스크 I/O |
| `-n DEV` | 네트워크 인터페이스 |
| `-n TCP` | TCP 통계 |
| `-q` | 런 큐/부하 평균 |

## 관측의 관측 (Observer Effect)

메트릭 수집 자체에 CPU 사이클이 소비된다. BPF 기반 도구는 커널에서 집계하여 이 오버헤드를 최소화한다.

## 관련 페이지

- [[perf]] — Linux 표준 프로파일러
- [[ftrace]] — Linux 표준 트레이서
- [[bpf]] — BPF 트레이싱 (BCC, bpftrace)
- [[systems-performance-methodology]] — 분석 방법론
