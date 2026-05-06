---
title: "Ftrace — Linux 트레이서"
type: tool
tags: [ftrace, tracing, kernel, function-graph, trace-cmd, linux]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# Ftrace — Linux 표준 트레이서

## 왜 Ftrace인가

Linux 커널 2.6.27(2008)부터 내장된 공식 트레이서. 추가 프론트엔드 없이 tracefs 파일시스템만으로 사용 가능 → 임베디드 환경에 적합. Steven Rostedt가 개발.

## 핵심 기능

| 기능 | 설명 |
|------|------|
| Function Profiler | 커널 함수 호출 통계 (호출 수, 평균 시간) |
| Function Tracing | 함수 호출별 상세 이벤트 출력 |
| function_graph | 자식 함수 계층 그래프 + 소요 시간 |
| Tracepoints | 커널 정적 계측 이벤트 |
| kprobes/uprobes | 커널/유저 동적 계측 |
| Hist Triggers | 사용자 정의 히스토그램 (Linux 4.7) |
| hwlat | 하드웨어 지연시간 감지 |

## tracefs 인터페이스

```bash
# 위치
/sys/kernel/debug/tracing/

# 주요 파일
current_tracer          # 현재 트레이서 (nop/function/function_graph)
set_ftrace_filter       # 트레이싱할 함수 선택
trace                   # 출력 (링 버퍼)
trace_pipe              # 라이브 스트림 (소비적)
kprobe_events           # kprobe 설정
uprobe_events           # uprobe 설정
events/                 # 트레이스포인트 제어
```

## Function Profiler

```bash
echo 'tcp*' > set_ftrace_filter
echo 1 > function_profile_enabled
sleep 10
echo 0 > function_profile_enabled
cat trace_stat/function0
# → Function, Hit, Time, Avg, s^2
```

## Function Graph Tracing

```bash
echo do_nanosleep > set_graph_function
echo function_graph > current_tracer
cat trace_pipe
```

지연시간 표시: `$`(>1s), `@`(>100ms), `*`(>10ms), `#`(>1ms), `!`(>100μs), `+`(>10μs)

## Tracepoints + Filter/Trigger

```bash
# block_rq_issue 활성화
echo 1 > events/block/block_rq_issue/enable

# 필터: 64K 초과만
echo 'bytes > 65536' > events/block/block_rq_insert/filter

# 트리거: 조건 충족 시 트레이싱 중단
echo 'traceoff if bytes > 65536' > events/block/block_rq_insert/trigger
```

## kprobes

```bash
echo 'p:myprobe do_nanosleep' >> kprobe_events
echo 1 > events/kprobes/myprobe/enable
cat trace_pipe
echo '-:myprobe' >> kprobe_events    # 삭제
```

인수 접근: `$arg1`, `$arg2` (Linux 4.20+) 또는 레지스터명 (`%di`, `%si`)

## Hist Triggers (Linux 4.7)

```bash
# PID별 syscall 히스토그램
echo 'hist:key=common_pid.execname' > events/raw_syscalls/sys_enter/trigger
sleep 10
cat events/raw_syscalls/sys_enter/hist

# 스택별 블록 I/O
echo 'hist:key=stacktrace' > events/block/block_rq_issue/trigger
```

### Synthetic Events (지연시간 계산)
두 이벤트 간 타임스탬프 차이를 계산하여 히스토그램 생성 — BPF 없이 복잡한 분석 가능.

## trace-cmd

Steven Rostedt의 Ftrace 프론트엔드.

```bash
trace-cmd record -p function -l 'tcp_*' sleep 10
trace-cmd report

trace-cmd record -p function_graph -g do_nanosleep sleep 10
trace-cmd report | cut -c 66-

trace-cmd record -e sched:sched_switch -T    # 커널 스택 포함
```

### KernelShark
trace-cmd 시각화 GUI. CPU별 타임라인 + 이벤트 테이블.

## hwlat (하드웨어 지연시간)

```bash
echo hwlat > current_tracer
cat trace_pipe
# → SMI, 하이퍼바이저 간섭 등 외부 하드웨어 지연 감지
```

## 관련 페이지

- [[perf]] — perf 도구 (PMC, 프로파일링)
- [[bpf]] — BPF 트레이싱
- [[observability]] — 관측 도구 개요
