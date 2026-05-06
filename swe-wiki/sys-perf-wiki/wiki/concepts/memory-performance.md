---
title: "메모리 성능"
type: concept
tags: [memory, virtual-memory, paging, swap, numa, oom]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# 메모리 성능

## 핵심 개념

### Virtual Memory
- 각 프로세스에 대형 선형 주소 공간 제공
- 물리 메모리 배치는 OS가 관리 → 소프트웨어 개발 단순화
- 메모리 오버서브스크립션(물리 이상 할당) 지원

### Paging
- **File system paging ("좋은 페이징")**: 메모리 매핑 파일 in/out. 깨끗하면 디스크 쓰기 불필요.
- **Anonymous paging = Swapping ("나쁜 페이징")**: 힙/스택 등 이름 없는 메모리를 swap 장치에 기록 → 애플리케이션 직접 성능 저하.

### Demand Paging
- `malloc()` → 실제 접근 시에만 물리 메모리 매핑 (lazy allocation)
- **Minor fault**: 디스크 I/O 없이 처리
- **Major fault**: 디스크 I/O 필요

### Working Set Size (WSS)
- 프로세스가 자주 사용하는 메모리 크기
- WSS가 CPU 캐시에 맞으면 매우 빠름
- WSS가 메인 메모리 초과 → swapping으로 급격한 성능 저하

### Overcommit
Linux는 물리+swap보다 더 많은 메모리 할당 허용 (기본). `vm.overcommit_memory`로 제어.

## 하드웨어 아키텍처

### DDR SDRAM

| 표준 | 피크 대역폭 |
|------|------------|
| DDR3-1600 | 12,800 MB/s |
| DDR4-3200 | 25,600 MB/s |
| DDR5-6400 | 51,200 MB/s |

Multichannel(dual/quad)로 대역폭 배가.

### NUMA (Non-Uniform Memory Access)
- CPU별 로컬 메모리, 타 CPU 메모리는 인터커넥트 경유 (높은 레이턴시)
- `numastat`으로 노드별 할당 통계 확인
- `numactl`로 CPU+메모리 동시 바인딩

## 소프트웨어 아키텍처

### 메모리 해제 순서 (가용 메모리 감소 시)

1. **Free list**: 미사용 페이지 즉시 할당
2. **Page cache 회수**: `swappiness`로 선호도 조정 (기본 60)
3. **Swapping**: kswapd가 비최근 사용 페이지를 swap에 기록
4. **Reaping**: slab allocator 캐시 해제
5. **OOM killer**: 희생 프로세스 선택 후 종료

### kswapd Watermarks

| Watermark | 동작 |
|-----------|------|
| high | kswapd 잠, 정상 상태 |
| low | kswapd 깨어나 스캔 시작 |
| min | foreground 직접 reclaim (성능 심각 저하) |

### Allocators

**커널 레벨:**
- **SLUB** (Linux 기본): slab 단순화, NUMA 최적화를 page allocator에 위임

**유저 레벨:**
- **glibc**: dlmalloc 기반
- **TCMalloc** (Google): 스레드별 캐시로 lock 경합 감소
- **jemalloc** (FreeBSD/Facebook): 다중 arenas, 단편화 감소

## USE Method 적용

| 메트릭 | 확인 방법 |
|--------|----------|
| Utilization | `free -m`, `vmstat`의 free/available |
| Saturation | `vmstat`의 si/so, `sar -B`의 pgscan, OOM kill (`dmesg`), PSI |
| Errors | `dmesg` (ECC 오류, OOM), `edac-utils` |

## 관측 도구

| 도구 | 핵심 메트릭 |
|------|-----------|
| `vmstat 1` | swpd, free, buff, cache, si/so |
| `free -m` | total, used, free, available |
| `sar -B` | pgscan(페이지 스캔), %vmeff(회수 효율) |
| `sar -W` | pswpin/s, pswpout/s |
| `slabtop -sc` | 커널 slab 캐시 사용 현황 |
| `numastat` | NUMA 노드별 hit/miss |
| `ps aux` | RSS, VSZ, %MEM |
| `pmap -x PID` | 프로세스 메모리 매핑 상세 |

### PSI (Pressure Stall Information)
```bash
cat /proc/pressure/memory
# some avg10=X.XX ...  # 일부 태스크 stall
# full avg10=X.XX ...  # 모든 태스크 stall
```

### perf (메모리 분석)
```bash
perf record -e page-faults -a -g -- sleep 60    # page fault flame graph
perf stat -e 'kmem:*' -a -I 1000               # kmem 이벤트 통계
```

### BPF 도구
```bash
drsnoop         # direct reclaim 추적 (레이턴시 포함)
wss PID         # Working Set Size 추정 (실험적)
memleak         # 미해제 메모리 추적
```

## 튜닝

### 주요 sysctl

| 튜너블 | 기본 | 설명 |
|--------|------|------|
| `vm.swappiness` | 60 | 0=앱 메모리 최대 유지, 100=적극적 swap |
| `vm.dirty_ratio` | 20 | write-back 트리거 비율 |
| `vm.min_free_kbytes` | 동적 | 최소 여유 메모리 |
| `vm.overcommit_memory` | 0 | 0=휴리스틱, 1=항상, 2=불허 |
| `vm.vfs_cache_pressure` | 100 | inode/dentry 캐시 회수 압력 |

### Huge Pages
큰 페이지(2MB, 1GB) → TLB 커버리지 증가 → TLB miss 감소.
- **THP (Transparent Huge Pages)**: 앱 수정 없이 자동 승격/강등

### NUMA Binding
```bash
numactl --membind=0 --physcpubind=0 PID  # CPU+메모리 동시 바인딩
```

### cgroups 메모리 제어
- `memory.limit_in_bytes`: 최대 메모리
- `memory.memsw.limit_in_bytes`: 최대 메모리+swap
- `memory.oom_control`: OOM killer 제어

## 관련 페이지

- [use-method](use-method.md) — USE 방법론
- [cpu-performance](cpu-performance.md) — CPU 캐시와 메모리 stall 관계
- [os-kernel](os-kernel.md) — 커널 메모리 관리
- [cloud-performance](cloud-performance.md) — 가상화 환경 메모리 오버헤드
