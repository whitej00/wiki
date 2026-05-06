---
title: "운영체제 커널과 성능"
type: concept
tags: [os, kernel, linux, syscall, scheduling, virtual-memory]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# 운영체제 커널과 성능

## 왜 커널을 이해해야 하는가

성능 분석가는 시스템 콜, 인터럽트, 스케줄링, 가상 메모리 등 커널 내부를 이해해야 성능 도구의 출력을 올바르게 해석할 수 있다.

## 커널 모드 vs 유저 모드

- **커널 모드(ring 0)**: 장치와 메모리에 직접 접근 가능. 커널 코드 실행.
- **유저 모드(ring 3)**: 시스템 콜을 통해서만 커널 서비스 요청.
- 전환 비용: 모드 스위치는 컨텍스트 스위치보다 가볍지만 빈번하면 오버헤드.

## 시스템 콜 (System Calls)

사용자 프로그램이 커널 서비스를 요청하는 인터페이스. `read(2)`, `write(2)`, `open(2)`, `close(2)`, `fork(2)`, `exec(2)` 등.

```
사용자 코드 → libc 래퍼 → syscall 명령어 → 커널 진입 → 처리 → 반환
```

성능 관점:
- 시스템 콜 자체의 CPU 오버헤드 (모드 전환, 인수 복사)
- 블로킹 시스템 콜은 스레드를 sleep 상태로 전환 → off-CPU 시간 증가
- `strace`로 추적 가능하지만 ptrace 기반이라 오버헤드 큼 → [bpf](../tools/bpf.md) 기반 `syscount` 권장

## 인터럽트

- **하드웨어 인터럽트**: 장치가 CPU에 이벤트 통지 (디스크 I/O 완료, 네트워크 패킷 도착)
- **소프트웨어 인터럽트(softirq)**: 커널이 지연 처리를 위해 사용
- 인터럽트 처리는 현재 스레드를 선점하므로 스케줄러 레이턴시에 영향

## 프로세스와 스레드

- **프로세스**: 독립적 주소 공간, `fork(2)`/`clone(2)`로 생성
- **스레드**: 프로세스 내 실행 단위, 주소 공간 공유
- Linux에서는 프로세스와 스레드 모두 `task_struct`로 관리
- 짧은 수명 프로세스는 `top`에서 보이지 않을 수 있다 → `execsnoop`으로 추적

## 스케줄러

→ [cpu-performance](cpu-performance.md) 참조

Linux 주요 스케줄링 클래스:
- **RT**: 실시간, 고정 우선순위 (0~99)
- **CFS**: Linux 2.6.23 기본, red-black tree, 공평한 시간 배분
- **Deadline**: EDF(Earliest Deadline First), Linux 3.14

## 가상 메모리

→ [memory-performance](memory-performance.md) 참조

- 각 프로세스에 대형 선형 주소 공간 제공
- MMU가 가상→물리 주소 변환, TLB가 변환 캐시
- demand paging: 실제 접근 시에만 물리 메모리 매핑

## 파일 시스템

→ [filesystem-performance](filesystem-performance.md) 참조

VFS(Virtual File System)가 서로 다른 파일 시스템 유형을 단일 인터페이스로 추상화.

## 캐싱

커널은 여러 레벨의 캐시를 관리:
- Page cache (파일 시스템 캐시)
- Dentry cache, inode cache
- Buffer cache
- Slab allocator cache

## Linux 커널 주요 성능 기능 변천

| 버전 | 주요 기능 |
|------|----------|
| 2.6 | O(1) 스케줄러, NPTL 스레드, epoll |
| 2.6.23 | CFS 스케줄러 |
| 3.2 | CFS bandwidth control |
| 3.13 | blk-mq (멀티큐 블록 I/O) |
| 4.9 | BPF tracing, cgroup v2 |
| 4.20 | PSI (Pressure Stall Information) |
| 5.1 | io_uring (비동기 I/O) |
| 5.6 | BPF 기반 TCP 혼잡 제어 |

## 특수 주제

### KPTI (Meltdown)
Meltdown 취약점 대응으로 커널/유저 페이지 테이블 분리. TLB 플러시 오버헤드 발생. PCID(Process Context ID) 지원 프로세서에서 오버헤드 감소.

### Extended BPF
→ [bpf](../tools/bpf.md) 참조. 2013년 이후 확장되어 일반 목적 인-커널 실행 환경이 됨.

### systemd
Linux의 init 시스템. cgroups 활용, 서비스 의존성 관리.

## 관련 페이지

- [cpu-performance](cpu-performance.md) — 스케줄러 상세
- [memory-performance](memory-performance.md) — 가상 메모리 상세
- [observability](observability.md) — 관측 도구 개요
- [bpf](../tools/bpf.md) — BPF 상세
