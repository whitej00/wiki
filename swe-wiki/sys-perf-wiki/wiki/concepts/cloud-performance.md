---
title: "클라우드 컴퓨팅 성능"
type: concept
tags: [cloud, virtualization, container, kubernetes, firecracker]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# 클라우드 컴퓨팅 성능

## 가상화 유형 비교

| 속성 | Hardware VM | Container | Lightweight VM |
|------|-------------|-----------|----------------|
| CPU 성능 | 높음 | 높음 | 높음 |
| 초기화 시간 | 수 분 | 밀리초 | ~100ms |
| 메모리 오버헤드 | 높음 (커널+VMM) | 낮음 | 매우 낮음 (~5MB) |
| 커널 분석 도구 | **동작** | **미동작** | **동작** |
| 보안 격리 | 높음 | 낮음 | 높음 |

## 하드웨어 가상화 (VM)

### 주요 하이퍼바이저
- **KVM**: Red Hat, Google Compute Engine
- **Xen**: AWS EC2 이전 하이퍼바이저
- **Nitro**: AWS 자체 개발(2017), KVM 기반 + 하드웨어 가속 → 근 bare-metal 성능

### CPU 오버헤드
- Hardware-assisted (VT-x/AMD-V): guest 특권 명령어가 VMM으로 트랩 → 낮은 오버헤드
- **VM exit** 이벤트가 핵심 오버헤드 지표: `perf kvm stat live`

### 메모리 오버헤드
- 2단계 변환: guest virtual → guest physical → host physical
- Intel EPT / AMD NPT로 TLB miss 시 하드웨어 page walk

### I/O 오버헤드
- SR-IOV: 게스트가 하드웨어 직접 접근 → 최저 I/O 오버헤드
- Nitro: 모든 주요 자원에 하드웨어 지원 → QEMU 불필요

### 관측
- **Steal time** (`vmstat`의 `st`): 다른 테넌트나 하이퍼바이저에 빼앗긴 CPU
- 게스트에서 모든 성능 도구 동작 (전용 커널 보유)

## OS 가상화 (Containers)

Linux 커널의 **namespaces** + **cgroups** + seccomp-bpf 조합.

### Namespace 격리

| Namespace | 격리 내용 |
|-----------|-----------|
| pid | 프로세스 가시성 |
| net | 네트워크 스택 |
| mnt | 파일시스템 마운트 |
| user | 사용자 ID |

### cgroups 리소스 제어
- **CPU**: shares(비율) + bandwidth(상한)
- **Memory**: limit_in_bytes, oom_control
- **blkio**: IOPS/처리량 제한

### 컨테이너의 관측성 한계

| 도구 | 호스트에서 | 컨테이너에서 |
|------|-----------|-------------|
| top | 전체 프로세스 | 컨테이너 프로세스만 |
| mpstat | 호스트 CPU | **호스트** CPU (오해 유발) |
| free | 호스트 메모리 | **호스트** 메모리 |
| perf/BPF | 완전 동작 | **미동작** 또는 제한적 |

**핵심 제한**: 컨테이너 내에서 커널 분석 도구(perf, BPF, dmesg)가 동작하지 않는다.

### CPU Bursting 문제
CFS shares로 유휴 CPU 버스팅 가능 → 테스트 시 좋은 성능 → 다른 테넌트 추가 후 **10배 성능 저하** 가능.

### Noisy Neighbor 현상
- CPU cache hit rate 저하 (캐시 오염)
- 커널 버퍼/캐시/큐/잠금 경쟁
- iptables 기반 컨테이너 네트워킹 오버헤드

## Kubernetes

### 성능 과제
- 스케줄링: CPU/메모리 requests/limits 기반 (블록 I/O 미지원)
- 네트워킹: CNI 플러그인 선택이 핵심
  - **Cilium** (BPF 기반) >> kube-proxy iptables 방식

### 리소스 설정
- `requests`: 최소 보장 (스케줄링 기준)
- `limits`: 최대 허용 (초과 시 throttle 또는 OOM kill)

## 경량 가상화 (Lightweight Virtualization)

컨테이너의 속도 + VM의 보안을 결합.

| 구현체 | 특징 |
|--------|------|
| **Firecracker** (AWS) | KVM 기반, ~100ms 부팅, ~5MB 오버헤드 |
| **Kata Containers** | ≤45ms 부팅 |
| **gVisor** (Google) | Go 유저스페이스 커널 |

Firecracker는 독립 커널 → **BPF, perf 등 모든 커널 도구 동작**.

## 관측 전략

1. **물리 리소스**: hypervisor 관리자 분석
2. **게스트 내부**: 게스트 접속 후 분석
3. **I/O 레이턴시 패턴**: `biosnoop` 시계열로 물리 경쟁/디바이스 문제 구분
4. **컨테이너 식별**: `docker stats`, `kubectl top`, cgroup 통계

## 관련 페이지

- [[use-method]] — USE 방법론
- [[cpu-performance]] — CPU 가상화 오버헤드
- [[network-performance]] — 네트워크 가상화
- [[bpf]] — BPF 기반 컨테이너 분석
