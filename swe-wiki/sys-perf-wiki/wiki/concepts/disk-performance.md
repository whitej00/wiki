---
title: "디스크 성능"
type: concept
tags: [disk, storage, iops, ssd, hdd, nvme, raid, io-scheduler]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# 디스크 성능

## 핵심 개념

### 시간 측정
```
I/O request time = I/O wait time (큐 대기) + I/O service time (처리)
```

### 타임 스케일

| 이벤트 | 레이턴시 |
|--------|---------|
| NVMe 캐시 히트 | 10~20 μs |
| 플래시 읽기 | 100~1,000 μs |
| HDD 순차 읽기 | ~1 ms |
| HDD 랜덤 읽기 (7,200rpm) | ~8 ms |

### Utilization과 Saturation
- **60% 이용률**부터 큐잉으로 인한 성능 저하 시작 (큐잉 이론)
- **100% 이용률** = 즉각적 성능 문제
- 가상 디스크 100%가 실제 포화를 의미하지 않을 수 있음 (여러 물리 디스크)

### I/O Wait
CPU가 유휴 + 디스크 I/O 대기 스레드 존재. 다른 CPU 작업이 추가되면 I/O wait 감소하지만 디스크 병목은 동일 → 애플리케이션 블로킹 시간이 더 신뢰할 수 있는 지표.

## 디스크 유형

### HDD (자기 회전 디스크)
- Seek time + Rotation latency가 성능 결정
- 순차 I/O >> 랜덤 I/O
- Short-stroking: 외부 트랙만 사용, 탐색 범위 감소

### SSD (Solid-State Drive)
- seek/rotation 없음, 랜덤/순차 차이 미미
- 플래시 유형: SLC(50K~100K 사이클) > MLC > TLC > QLC(~1K 사이클)
- Write amplification: 블록 크기보다 작은 쓰기 시 read-modify-write
- TRIM 명령으로 불필요 블록 통지 → 쓰기 성능 유지

### NVMe
- PCIe 버스 직접 연결, < 20μs 레이턴시
- 64K 명령 큐 지원 → 기존 SATA(32) 대비 압도적 IOPS

## Linux I/O 스택

```
VFS → 파일 시스템 → Page Cache → 블록 디바이스 → I/O 머징 → I/O 스케줄러 → 드라이버 → 디스크
```

### I/O 스케줄러 (blk-mq)

| 스케줄러 | 특징 |
|---------|------|
| **none** | 스케줄링 없음 (NVMe에 적합) |
| **mq-deadline** | 레이턴시 데드라인 보장, 기아 방지 |
| **BFQ** | 프로세스별 대역폭+시간 예산, cgroups 지원 |
| **Kyber** | 타겟 레이턴시 기반 동적 큐 조정 (Netflix 기본) |

## USE Method 적용

| 메트릭 | 확인 방법 |
|--------|----------|
| Utilization | `iostat -xz 1`의 %util |
| Saturation | `iostat`의 aqu-sz, PSI `/proc/pressure/io` |
| Errors | `smartctl`, `dmesg` |

## 관측 도구

### iostat (핵심 도구)
```bash
iostat -sxz 1
```

| 컬럼 | 의미 |
|------|------|
| **await** | 평균 I/O 응답 시간 (ms) — **가장 중요** |
| **r_await** | 읽기 평균 응답 시간 — 애플리케이션에 가장 직접적 |
| **aqu-sz** | 평균 큐+장치 요청 수 |
| **areq-sz** | 평균 요청 크기 (작으면 랜덤, 크면 순차) |
| **%util** | 장치 이용률 |
| **%rrqm/%wrqm** | 머징 비율 (높으면 순차 워크로드) |

### BPF 기반 도구

| 도구 | 용도 |
|------|------|
| `biolatency(8)` | 디스크 I/O 레이턴시 히스토그램. `-F`로 I/O 플래그별 분리 |
| `biosnoop(8)` | 개별 I/O 추적 (PID, DISK, SECTOR, BYTES, LAT) |
| `biotop(8)` | 프로세스별 디스크 I/O 요약 |
| `biostacks(8)` | I/O 초기화 스택 추적 포함 |

### blktrace
블록 디바이스 I/O를 단계별로 추적. `btrace /dev/sda`로 간단 사용.

## RAID

| RAID | 성능 특성 |
|------|----------|
| 0 (stripe) | 최고 성능 (중복성 없음) |
| 1 (mirror) | 읽기 우수, 쓰기는 가장 느린 디스크 기준 |
| 10 | RAID-0 + RAID-1 결합, 확장 가능 |
| 5 | 쓰기 poor (read-modify-write 패리티 계산) |

## 시각화

- **레이턴시 히트맵**: X=시간, Y=레이턴시, 색=I/O 수. Brendan Gregg 발명. 다중 분포, 패턴 변화 시각화.
- **오프셋 히트맵**: Y=디스크 오프셋. 순차(진한 선) vs 랜덤(흐린 구름) 패턴.

## 튜닝

| 항목 | 방법 |
|------|------|
| I/O 스케줄러 | `/sys/block/*/queue/scheduler` |
| Read-ahead | `/sys/block/*/queue/read_ahead_kb` |
| ionice | 클래스 0~3 (idle이 백업에 적합) |
| cgroups blkio | IOPS/처리량 제한 |

## 관련 페이지

- [filesystem-performance](filesystem-performance.md) — 파일 시스템 레이어
- [use-method](use-method.md) — USE 방법론
- [cloud-performance](cloud-performance.md) — 가상 디스크 성능
- [benchmarking](benchmarking.md) — 디스크 벤치마킹
