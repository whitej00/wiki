---
title: "perf — Linux 프로파일러"
type: tool
tags: [perf, profiling, tracing, pmc, flame-graph, linux]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# perf — Linux 표준 프로파일러

## 왜 perf인가

Linux 커널에 포함된 공식 성능 분석 도구. PMC(하드웨어 카운터), 트레이스포인트, kprobes, uprobes, USDT를 모두 지원한다.

## 주요 서브커맨드

| 커맨드 | 설명 |
|--------|------|
| `stat` | 이벤트 카운팅 (PMC 통계, IPC 등) |
| `record` | 이벤트를 perf.data에 저장 |
| `report` | perf.data 요약 (TUI 또는 STDIO) |
| `script` | perf.data 이벤트 상세 출력 |
| `trace` | 시스콜 라이브 트레이서 (strace 대체) |
| `top` | 실시간 CPU 프로파일 |
| `list` | 사용 가능한 이벤트 목록 |
| `probe` | 동적 프로브(kprobe/uprobe) 정의 |

## 이벤트 유형

| 유형 | 예시 |
|------|------|
| Hardware | cycles, instructions, cache-misses, branch-misses |
| Software | context-switches, cpu-clock, page-faults |
| Tracepoint | sched:sched_switch, block:block_rq_issue |
| kprobe | 커널 함수 동적 계측 |
| uprobe | 유저 함수 동적 계측 |
| USDT | 유저 정적 계측 (Node.js, Java 등) |

## 핵심 One-Liners

### PMC 통계 (IPC 확인)
```bash
perf stat -a -- sleep 5
# → cycles, instructions, IPC 자동 계산
```

### CPU Profiling + Flame Graph
```bash
perf record -F 99 -a -g -- sleep 30
perf script report flamegraph    # Linux 5.8+
```

### 시스콜 트레이싱
```bash
perf trace -p $(pgrep mysqld)    # strace보다 낮은 오버헤드
```

### 블록 I/O 트레이싱
```bash
perf record -e block:block_rq_issue -a -g sleep 10
perf script
```

### kprobe/uprobe
```bash
# kprobe
perf probe --add tcp_sendmsg
perf record -e probe:tcp_sendmsg -a sleep 5

# uprobe (libc fopen)
perf probe -x /lib/x86_64-linux-gnu/libc.so.6 --add fopen
```

## perf stat 상세

```bash
perf stat -e cycles,instructions -a -- sleep 5
# → IPC(insn per cycle) "shadow statistic" 자동 계산

perf stat -e LLC-loads,LLC-load-misses -a -- sleep 5
# → LLC hit rate 자동 계산

perf stat -I 1000 -e sched:sched_switch -a
# → 1초 간격 인터벌 통계
```

## perf record 스택 워킹

| 옵션 | 방법 | 장단점 |
|------|------|--------|
| `-g` (기본) | Frame pointer | 빠르지만 -fomit-frame-pointer에서 깨짐 |
| `--call-graph dwarf` | DWARF debuginfo | 정확, 파일 크기 큼 |
| `--call-graph lbr` | Intel LBR | 낮은 오버헤드, 최대 16~32 프레임 |

## perf trace

strace(1)보다 낮은 오버헤드로 시스콜 트레이싱:
```bash
perf trace                           # 전체 시스템
perf trace -e syscalls:*enter_mmap --filter='flags==SHARED'  # 필터
```

## KVM 분석

```bash
perf kvm stat live    # VM exit 유형별 통계
```

## 관련 페이지

- [[cpu-performance]] — CPU 프로파일링
- [[ftrace]] — Linux 표준 트레이서
- [[bpf]] — BPF 트레이싱
- [[observability]] — 관측 도구 개요
