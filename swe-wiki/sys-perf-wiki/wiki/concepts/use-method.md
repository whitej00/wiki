---
title: "USE 방법론"
type: concept
tags: [performance, methodology, use-method, troubleshooting]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# USE 방법론 (Utilization, Saturation, Errors)

Brendan Gregg가 개발한 시스템 성능 병목 식별 방법론.

## 왜 USE인가

성능 문제의 대부분은 리소스 병목에서 비롯된다. USE는 **모든 리소스를 체계적으로 점검**하여 빠뜨림 없이 병목을 찾는다.

## 핵심 원칙

> 모든 리소스에 대해 **Utilization**, **Saturation**, **Errors**를 확인하라.

### 정의

| 메트릭 | 의미 | 확인 우선순위 |
|--------|------|-------------|
| **Errors** | 오류 이벤트 수 | 1순위 (가장 먼저) |
| **Saturation** | 처리 불가로 대기 중인 작업 | 2순위 |
| **Utilization** | 리소스 사용 비율 | 3순위 |

오류를 먼저 확인하는 이유: 객관적이고 해석이 빠르다.

## 리소스 목록

| 리소스 | 설명 |
|--------|------|
| CPUs | 소켓, 코어, 하드웨어 스레드 |
| Main memory | DRAM |
| Network interfaces | 이더넷, InfiniBand |
| Storage devices | 디스크, 스토리지 어댑터 |
| Accelerators | GPU, TPU, FPGA |
| Controllers | 스토리지, 네트워크 컨트롤러 |
| Interconnects | CPU, 메모리, I/O 인터커넥트 |

## 리소스별 체크리스트

### CPU
| 메트릭 | 확인 방법 |
|--------|----------|
| Utilization | `mpstat -P ALL 1` (%usr + %sys), `sar -u` |
| Saturation | 런 큐 길이 (`vmstat`의 `r`), 스케줄러 레이턴시 (`runqlat`), PSI (`/proc/pressure/cpu`) |
| Errors | `perf stat` (하드웨어 오류), `dmesg` (MCE) |

### Memory
| 메트릭 | 확인 방법 |
|--------|----------|
| Utilization | `free -m`, `vmstat`의 free/available |
| Saturation | swap 활동 (`vmstat`의 si/so), page scanning (`sar -B`의 pgscan), OOM kill (`dmesg`) |
| Errors | ECC 오류 (`dmidecode`, `edac-utils`), `dmesg` |

### Network Interface
| 메트릭 | 확인 방법 |
|--------|----------|
| Utilization | `sar -n DEV 1` (rxkB/s, txkB/s / 최대 대역폭), `nicstat` (%Util) |
| Saturation | `ip -s link`의 overruns, TCP 재전송 (`nstat`의 TcpRetransSegs) |
| Errors | `ip -s link`의 errors, drops |

### Storage Device I/O
| 메트릭 | 확인 방법 |
|--------|----------|
| Utilization | `iostat -xz 1`의 %util |
| Saturation | `iostat`의 aqu-sz (대기 큐), PSI (`/proc/pressure/io`) |
| Errors | `smartctl`, `/var/log/messages`, `dmesg` |

## 해석 가이드

| 상태 | 해석 |
|------|------|
| Utilization 100% | 병목 신호. 포화와 그 영향을 함께 확인 |
| **Utilization > 60%** | 문제 가능성. 짧은 100% 버스트를 숨길 수 있음 (큐잉 이론에서 60%부터 응답 시간 급증) |
| Saturation > 0 | 문제. 어떤 정도의 포화도 레이턴시 증가를 의미 |
| Errors 증가 | 즉시 조사 필요 |

### 단기 급등(Burst) 주의

5분 평균 활용률 30%이더라도 **초 단위 100% 구간**이 숨어있을 수 있다. 가능하면 짧은 인터벌(1초)로 측정하라.

## 소프트웨어 리소스 적용

| 리소스 | Utilization | Saturation | Errors |
|--------|------------|-----------|--------|
| Mutex locks | 잠금 유지 시간 | 대기 스레드 수 | — |
| Thread pools | 작업 처리 시간 | 대기 요청 수 | — |
| Process capacity | 현재 사용량 | 할당 대기 | 할당 실패 |
| File descriptors | 현재 사용/최대 | 할당 대기 | 한도 초과 |

## 마이크로서비스에서의 USE (Netflix)

Netflix Atlas 모니터링에서 각 마이크로서비스에 대해:
- **Utilization**: 클러스터 평균 CPU 활용률
- **Saturation**: p99 레이턴시 − 평균 레이턴시
- **Errors**: 요청 오류 수

## 관련 페이지

- [systems-performance-methodology](systems-performance-methodology.md) — 전체 방법론 개요
- [cpu-performance](cpu-performance.md) — CPU USE 적용
- [memory-performance](memory-performance.md) — Memory USE 적용
- [disk-performance](disk-performance.md) — Disk USE 적용
- [network-performance](network-performance.md) — Network USE 적용
