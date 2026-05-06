---
title: "BPF — 프로그래밍 가능한 트레이싱"
type: tool
tags: [bpf, ebpf, bcc, bpftrace, tracing, observability, linux]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# BPF — 프로그래밍 가능한 트레이싱

## 왜 BPF인가

Extended BPF(eBPF)는 커널 내에서 직접 필터링/집계/지연시간 계산을 수행한다. 이벤트를 유저 공간으로 덤프하는 대신 **커널 컨텍스트에서 처리** → 프로덕션에서도 사용 가능한 낮은 오버헤드.

## BCC vs bpftrace

| 특성 | BCC | bpftrace |
|------|-----|---------|
| 용도 | 복잡한 옵션 지원 도구 | 단순 원라이너/탐색 |
| 언어 | Python/Lua + C (커널) | bpftrace (awk 유사) |
| 난이도 | 어려움 | 쉬움 |
| 평균 프로그램 | 228줄 | 28줄 |
| 도구 수 | 80+ | 30+ |

**BCC = C 프로그래밍, bpftrace = 쉘 스크립팅**

## BCC 주요 도구

### 범용 도구

| 도구 | 용도 |
|------|------|
| `funccount` | 커널/유저 함수 호출 카운팅 |
| `funclatency` | 함수 지연시간 히스토그램 |
| `stackcount` | 이벤트별 스택 트레이스 카운팅 |
| `trace` | 필터를 사용한 임의 함수 트레이싱 |
| `argdist` | 함수 파라미터 히스토그램 |

### CPU
| 도구 | 용도 |
|------|------|
| `profile` | 타이머 기반 CPU 프로파일러 (49/99 Hz) |
| `runqlat` | 런 큐 레이턴시 히스토그램 |
| `runqlen` | 런 큐 길이 분포 |
| `cpudist` | on/off-CPU 시간 분포 |
| `offcputime` | off-CPU 시간 스택 요약 |

### Memory
| 도구 | 용도 |
|------|------|
| `memleak` | 미해제 메모리 추적 |
| `drsnoop` | direct reclaim 추적 |

### File System / Disk
| 도구 | 용도 |
|------|------|
| `opensnoop` | open(2) 추적 |
| `filetop` | 파일별 I/O 상위 목록 |
| `cachestat` | 페이지 캐시 히트/미스율 |
| `ext4dist` / `xfsdist` | FS 연산 레이턴시 히스토그램 |
| `biolatency` | 블록 I/O 레이턴시 히스토그램 |
| `biosnoop` | 개별 블록 I/O 추적 |
| `biotop` | 프로세스별 블록 I/O 요약 |

### Network
| 도구 | 용도 |
|------|------|
| `tcplife` | TCP 세션 수명/처리량 |
| `tcpretrans` | TCP 재전송 추적 |
| `tcptop` | 호스트별 TCP 전송량 |

### Application
| 도구 | 용도 |
|------|------|
| `execsnoop` | 새 프로세스 추적 |
| `syscount` | 시스콜 카운트 |

## bpftrace

### 프로그램 구조
```
probes /filter/ { actions }
```

### 프로브 유형

| Type | 단축 | 설명 |
|------|------|------|
| `tracepoint` | `t` | 커널 정적 계측 |
| `kprobe` | `k` | 커널 함수 진입 |
| `kretprobe` | `kr` | 커널 함수 반환 |
| `kfunc` | `f` | BTF 기반 저오버헤드 |
| `uprobe` | `u` | 유저 함수 진입 |
| `uretprobe` | `ur` | 유저 함수 반환 |
| `usdt` | `U` | 유저 USDT |
| `profile` | `p` | 전체 CPU 타이머 |
| `interval` | `i` | 주기적 리포팅 |

### 핵심 One-Liners

**CPU:**
```bash
bpftrace -e 'profile:hz:99 { @[comm] = count(); }'         # 실행 프로세스 샘플링
bpftrace -e 'profile:hz:49 { @[kstack, ustack, comm] = count(); }' # 스택 포함
```

**파일 시스템:**
```bash
bpftrace -e 't:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args->filename)); }'
bpftrace -e 'kprobe:vfs_* { @[probe] = count(); }'         # VFS 카운트
```

**디스크:**
```bash
bpftrace -e 't:block:block_rq_issue { @bytes = hist(args->bytes); }'
```

**네트워크:**
```bash
bpftrace -e 'k:tcp_sendmsg { @send_bytes = hist(arg2); }'
bpftrace -e 't:tcp:tcp_retransmit_* { @[ntop(2,args->saddr)] = count(); }'
```

**메모리:**
```bash
bpftrace -e 'software:page-fault:1 { @[comm, pid] = count(); }'
```

### 지연시간 측정 패턴 (vfs_read 예시)
```python
kprobe:vfs_read {
    @start[tid] = nsecs;
}
kretprobe:vfs_read /@start[tid]/ {
    @us = hist((nsecs - @start[tid]) / 1000);
    delete(@start[tid]);
}
```

### Map 함수

| 함수 | 설명 |
|------|------|
| `count()` | 카운팅 |
| `sum(n)` | 합산 |
| `hist(n)` | power-of-two 히스토그램 |
| `lhist(n,min,max,step)` | 선형 히스토그램 |
| `avg(n)` | 평균 |
| `min(n)` / `max(n)` | 최솟값/최댓값 |

### 내장 변수

| 변수 | 의미 |
|------|------|
| `pid`, `tid` | 프로세스/스레드 ID |
| `comm` | 프로세스명 |
| `nsecs` | 나노초 타임스탬프 |
| `cpu` | CPU ID |
| `kstack` / `ustack` | 커널/유저 스택 |
| `arg0`~`argN` | 함수 인수 |
| `retval` | 반환값 |
| `curtask` | 현재 task_struct |

## 컨테이너에서의 BPF

컨테이너(OS 가상화) 내에서는 BPF 도구가 **동작하지 않는다** (커널 공유). 호스트에서 실행하고 UTS namespace의 nodename으로 컨테이너를 구분:

```
$task = (struct task_struct *)curtask;
$uts = $task->nsproxy->uts_ns->name;
@[str($uts.nodename), comm] = count();
```

경량 가상화(Firecracker)에서는 독립 커널이므로 BPF 완전 동작.

## 도구 선택 가이드

| 목적 | 도구 |
|------|------|
| 하드웨어 PMC 분석 | [perf](perf.md) stat/record |
| CPU Flame Graph | [perf](perf.md) record + script |
| 커널 함수 그래프 | [ftrace](ftrace.md) function_graph |
| 커스텀 히스토그램 | bpftrace, Ftrace hist triggers |
| 복잡한 프로덕션 도구 | BCC (Python/C) |
| 빠른 원라이너 탐색 | bpftrace |
| 임베디드/최소 의존성 | perf-tools (shell + awk) |

## 관련 페이지

- [perf](perf.md) — PMC와 프로파일링
- [ftrace](ftrace.md) — Ftrace 트레이서
- [observability](../concepts/observability.md) — 관측 도구 개요
- [cpu-performance](../concepts/cpu-performance.md) — CPU 분석에서의 BPF 활용
- [cloud-performance](../concepts/cloud-performance.md) — 컨테이너에서의 BPF 제한
