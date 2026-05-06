---
title: "애플리케이션 성능"
type: concept
tags: [application, performance, profiling, concurrency, gc]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# 애플리케이션 성능

## 왜 애플리케이션 레벨에서 분석하는가

성능 튜닝은 **작업이 수행되는 곳에 가장 가까운 위치**에서 가장 효과적이다. 애플리케이션 레이어에서 불필요한 쿼리를 제거하면 20x 향상이 가능하지만, 스토리지 레이어 튜닝은 ~20% 수준.

## 성능 기법

### I/O Size 선택
- 너무 작은 I/O: 시스템 콜 오버헤드 증가
- 너무 큰 I/O: 불필요한 데이터 전송
- 최적 I/O 크기는 워크로드에 따라 다름

### 캐싱
가장 효과적인 성능 기법 중 하나. 중복 계산이나 I/O를 메모리에서 처리.

### 버퍼링
소규모 I/O를 모아 대규모 I/O로 병합. 시스템 콜 횟수 감소.

### Concurrency와 Parallelism
- **Concurrency**: 여러 작업을 동시에 관리 (I/O 멀티플렉싱)
- **Parallelism**: 여러 CPU에서 동시 실행

| 모델 | 특징 |
|------|------|
| Process pool | fork()로 워커 프로세스 생성 |
| Thread pool | pthread로 워커 스레드 생성 |
| Event-driven | epoll/kqueue 기반 논블로킹 I/O |

### Non-Blocking I/O
`O_NONBLOCK` 또는 `io_uring`(Linux 5.1)으로 블로킹 없이 I/O 처리.

### Processor Binding
`taskset`이나 `numactl`로 특정 CPU에 바인딩 → 캐시 히트율 향상.

## 프로그래밍 언어 특성

| 유형 | 예시 | 성능 특성 |
|------|------|----------|
| Compiled | C, C++, Go, Rust | 직접 기계어 실행, 최고 성능 |
| Interpreted | Python, Ruby, Perl | 인터프리터 오버헤드, 관측 용이 |
| VM | Java (JVM), C# (.NET) | JIT 컴파일로 최적화, GC 오버헤드 |

### Garbage Collection (GC)
- **Stop-the-world**: 전체 애플리케이션 일시 정지 → 레이턴시 스파이크
- **Concurrent GC**: 일부 또는 전체 GC를 애플리케이션과 병렬 실행
- GC 튜닝은 힙 크기, GC 알고리즘 선택이 핵심

## 분석 방법론

### CPU Profiling
가장 기본적이고 효과적인 방법. 99Hz 타이머 기반 샘플링 후 Flame Graph로 시각화.

```bash
perf record -F 99 -a -g -- sleep 30
perf script report flamegraph
```

### Off-CPU Analysis
스레드가 CPU에서 실행되지 않는 시간(블로킹 I/O, 락, sleep) 분석.

```bash
offcputime -f 5   # BCC 도구, 5초간 off-CPU 시간 수집
```

On-CPU + Off-CPU = **전체 스레드 시간**. 둘 다 분석해야 완전한 그림.

### Thread State Analysis

스레드 상태를 시간별로 분석:
1. **Executing** (on-CPU)
2. **Runnable** (런 큐 대기)
3. **Swapping** (스왑 I/O 대기)
4. **Disk I/O** (디스크 대기)
5. **Network I/O** (네트워크 대기)
6. **Sleeping** (타이머/조건 대기)
7. **Lock** (락 대기)
8. **Idle** (작업 없음)

### Syscall Analysis
시스템 콜 유형, 빈도, 레이턴시를 분석하여 병목 식별.

```bash
syscount -P         # BCC: 프로세스별 시스콜 카운트
strace -c command   # 시스콜 요약 (오버헤드 높음)
```

### Lock Analysis
뮤텍스/스핀락 경합 분석. CPU 프로파일에서 스핀 경로, off-CPU에서 블로킹 경로 확인.

### Distributed Tracing
마이크로서비스 간 요청 흐름 추적. 각 서비스의 레이턴시 기여도 파악.

## 주요 도구

| 도구 | 용도 |
|------|------|
| `perf record` | CPU 프로파일링 |
| `profile(8)` | BPF 기반 CPU 프로파일러 (낮은 오버헤드) |
| `offcputime(8)` | Off-CPU 시간 분석 |
| `strace` | 시스콜 트레이싱 (높은 오버헤드) |
| `execsnoop(8)` | 새 프로세스 추적 |
| `syscount(8)` | 시스콜 카운트 |

## Gotchas

### Missing Symbols
컴파일러 최적화나 스트립된 바이너리에서 함수명이 `[unknown]`으로 표시. 해결: debuginfo 패키지 설치, `-fno-omit-frame-pointer` 컴파일.

### Missing Stacks
프레임 포인터 최적화로 스택이 깨짐. 해결: `--call-graph dwarf` 또는 `--call-graph lbr` 사용.

## 관련 페이지

- [[cpu-performance]] — CPU 프로파일링 상세
- [[observability]] — 관측 도구
- [[perf]] — perf 도구 상세
- [[bpf]] — BPF 기반 분석
