---
title: "CPU 성능"
type: concept
tags: [cpu, performance, scheduling, profiling, flame-graph, ipc]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# CPU 성능

## 핵심 개념

### 클럭과 명령어
- 4 GHz CPU = 초당 40억 사이클. 단, 메모리 stall 중에는 유용한 작업 불가.
- **IPC (Instructions Per Cycle)**: CPU 효율의 고수준 지표
  - IPC < 0.2 → 메모리 stall이 심각
  - IPC > 1.0 → 파이프라인/수퍼스칼라 효과로 효율적
- **CPI = 1/IPC**: 명령어당 사이클 수

### SMT (Simultaneous Multithreading)
- 하나의 코어에서 여러 스레드 실행 (Intel Hyper-Threading: 코어당 2 스레드)
- stall cycle이 많은 워크로드가 SMT에서 더 잘 동작

### Utilization
- CPU idle 스레드를 실행하지 않는 시간의 비율
- 높은 이용률이 반드시 문제는 아님 — 단, 메모리 stall도 이용률에 포함
- User time vs Kernel time: 연산 집약적 = ~99/1, I/O 집약적 = ~70/30

### Saturation
- CPU 100% → 런 큐에서 대기하는 스레드 → 전체 성능 저하
- 클라우드에서는 물리적 CPU가 100%가 아니어도 **CPU quota 도달**로 포화 발생

### Priority Inversion
낮은 우선순위 스레드가 리소스를 잡고 있어 높은 우선순위 스레드가 블록. 해결: Priority Inheritance.

## 하드웨어 아키텍처

### CPU 캐시 계층

| 캐시 | 크기 (일반) | 레이턴시 |
|------|------------|---------|
| L1 I-cache | 32~64 KB | 수 사이클 |
| L1 D-cache | 32~64 KB | 수 사이클 |
| L2 | 256 KB~1 MB | ~12 사이클 |
| L3 (LLC) | 수~수십 MB | ~40 사이클 |
| Main Memory | — | ~240 사이클 (~60ns) |

- **Set associativity**: 4-way 등으로 캐시 라인 배치 위치 선택
- **Cache line**: 64 bytes (x86)
- **Cache coherency**: 멀티 CPU 캐시 일관성 유지 프로토콜

### TLB (Translation Lookaside Buffer)
- MMU의 주소 변환 캐시
- TLB miss → 메인 메모리의 page table 참조 (수백 사이클)
- Huge pages로 TLB 커버리지 확대 → TLB miss 감소

### CPU Interconnects
- Intel UPI (2017): 41.6 GB/s
- NUMA: CPU별 로컬 메모리, 타 CPU 메모리는 인터커넥트 경유 (higher latency)

### PMCs (Performance Monitoring Counters)
하드웨어 레지스터로 저수준 CPU 활동 측정. `perf stat`으로 접근.

## 소프트웨어: Linux 스케줄러

### 스케줄링 클래스

| 클래스 | 설명 |
|--------|------|
| **RT** | 고정 우선순위 (0~99), SCHED_FIFO/SCHED_RR |
| **CFS** | Linux 기본. Red-black tree, 공평한 시간 배분 (SCHED_NORMAL) |
| **Deadline** | EDF, runtime/period/deadline 3개 파라미터 (SCHED_DEADLINE) |

### CFS (Completely Fair Scheduler)
- `vruntime`(가상 실행 시간)이 가장 작은 스레드를 다음에 실행
- nice 값으로 가중치 조정 (-20 ~ +19)
- cgroup으로 CPU shares/bandwidth 제한

## USE Method 적용

| 메트릭 | 확인 방법 |
|--------|----------|
| Utilization | `mpstat -P ALL 1`, `sar -u` |
| Saturation | `vmstat`의 `r`, `runqlat(8)`, PSI `/proc/pressure/cpu` |
| Errors | `perf stat` (MCE), `dmesg` |

## 프로파일링

### CPU Profiling (99Hz 타이머 기반)
```bash
perf record -F 99 -a -g -- sleep 30
perf script report flamegraph    # Linux 5.8+
```

BPF 기반 (낮은 오버헤드):
```bash
profile -F 99 -af 30    # BCC, flamegraph 형식 출력
```

### Flame Graph 해석
- **X축**: 샘플 모집단 (시간 흐름이 아님, 알파벳순 정렬)
- **Y축**: 스택 깊이
- **너비**: on-CPU 시간 비율
- 상단에서 큰 "plateau" 찾기 → 단일 함수가 CPU를 많이 소비
- 인터랙티브: 클릭 줌, Ctrl-F 검색

### Cycle Analysis
PMC로 IPC 측정 → stall cycle 유형 분석:
```bash
perf stat -e cycles,instructions,LLC-load-misses -a -- sleep 5
```

## 관측 도구

| 도구 | 핵심 메트릭 |
|------|-----------|
| `uptime` | 1/5/15분 load average |
| `vmstat 1` | `r`(런큐), `us`/`sy`/`id`/`wa`/`st` |
| `mpstat -P ALL 1` | per-CPU %usr, %sys, %idle, %steal |
| `pidstat 1` | 프로세스별 CPU |
| `perf stat` | IPC, 캐시 미스, 분기 예측 실패율 |
| `runqlat(8)` | 런 큐 레이턴시 히스토그램 |
| `runqlen(8)` | 런 큐 길이 분포 |
| `softirqs(8)` / `hardirqs(8)` | 인터럽트별 CPU 시간 |

### PSI (Pressure Stall Information)
```bash
cat /proc/pressure/cpu
# some avg10=X.XX avg60=X.XX avg300=X.XX total=...
```

### Load Average
- Linux load average = CPU 수요 + **디스크 I/O** + 기타 (1993년 변경)
- 부하 = utilization + saturation

## 시각화

- **Utilization Heat Map**: X=시간, Y=이용률(0~100%), 색=해당 이용률의 CPU 수
- **Flame Graph**: CPU 프로파일 시각화
- **FlameScope**: Subsecond-offset heat map + Flame graph 결합 (Netflix)

## 튜닝

| 항목 | 방법 |
|------|------|
| 스케줄링 우선순위 | `nice -n 19 command`, `chrt`, `renice` |
| Scaling Governor | `performance` (최대 주파수) vs `powersave` |
| CPU Binding | `taskset -pc 7-10 PID`, `numactl` |
| Exclusive CPU Sets | cpusets로 CPU 그룹 전용 할당 |
| cgroups | CPU shares 또는 CFS bandwidth 제한 |

### 주요 sysctl 튜너블

| 튜너블 | 기본값 | 설명 |
|--------|--------|------|
| `kernel.sched_latency_ns` | 12,000,000 | 목표 선점 레이턴시 |
| `kernel.sched_migration_cost_ns` | 500,000 | 캐시 hot 판단 기준 |

## 관련 페이지

- [[use-method]] — USE 방법론
- [[perf]] — perf 도구 상세
- [[bpf]] — BPF 기반 CPU 도구
- [[memory-performance]] — 메모리 stall과 CPU 관계
- [[cloud-performance]] — 가상화 환경 CPU 오버헤드
