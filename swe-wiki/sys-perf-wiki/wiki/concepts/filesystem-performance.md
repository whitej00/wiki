---
title: "파일 시스템 성능"
type: concept
tags: [filesystem, cache, vfs, ext4, xfs, zfs, io]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# 파일 시스템 성능

## 왜 파일 시스템 레벨에서 분석하는가

파일 시스템 레이턴시는 애플리케이션 성능에 **직접적·비례적으로** 영향을 준다. OS가 전통적으로 디스크 수준 통계만 제공했으나, 이는 애플리케이션 성능과 무관한 경우가 많다. **파일 시스템 레이턴시 자체**를 측정해야 한다.

## 핵심 개념

### Logical I/O vs Physical I/O

1바이트 쓰기가 실제로 128KB 읽기 + 128KB 쓰기 + 메타데이터 쓰기를 발생시킬 수 있다.

| 유형 | 원인 |
|------|------|
| Deflated (축소) | 캐시 히트, write cancellation, 압축 |
| Inflated (팽창) | 메타데이터, 레코드 크기 반올림, 저널링(2배), RAID 패리티 |

### 캐싱 계층

| 캐시 | 설명 |
|------|------|
| Page cache | OS 페이지 캐시 (가장 중요) |
| Dentry cache | 디렉토리 엔트리→inode 매핑 |
| Inode cache | VFS inode 캐싱 |
| Buffer cache | 블록 디바이스 캐시 (page cache에 통합) |

### Prefetch / Read-Ahead
순차 읽기 패턴 감지 후 애플리케이션 요청 전에 미리 디스크에서 데이터 로드.

### Write-Back Caching
쓰기를 메모리에서 완료 처리 후 비동기 디스크 플러시. 성능 크게 향상, 전원 장애 위험.

### 연산의 불균등성
캐시 히트(~3μs)와 디스크 읽기(~수백μs)는 수십~수백 배 차이. 500 ops/s만으로는 의미 없다.

## I/O 스택

```
애플리케이션 → VFS → 파일 시스템 → Page Cache → 블록 디바이스 → 디스크
```

### VFS (Virtual File System)
서로 다른 파일 시스템을 단일 인터페이스로 추상화. 모든 파일 시스템에 대한 단일 계측 지점.

## 파일 시스템 유형

| 파일 시스템 | 특징 |
|------------|------|
| **ext4** (2008) | Extents 기반, delayed allocation, 저널링. Linux 기본 |
| **XFS** (1993) | Allocation Groups로 병렬 접근, 온라인 디프래그. Netflix Cassandra 사용 |
| **ZFS** (2005) | Pooled storage, COW, ARC(MRU+MFU), L2ARC, SLOG, 체크섬, 압축 |
| **btrfs** (2007) | COW B-tree, pooled storage, 스냅샷, 압축 |
| **tmpfs** | 메모리 기반 임시 파일 시스템 |

### ZFS ARC (Adaptive Replacement Cache)
MRU + MFU 병합 알고리즘. Ghost list로 최적 균형 자동 조절 → 높은 캐시 히트율.

## 방법론

### 레이턴시 분석
```
파일 시스템 시간 비율 = 100 × 총 블로킹 FS 레이턴시 / 트랜잭션 시간
```
예: 트랜잭션 200ms 중 FS 블로킹 180ms → 90% → 최대 10x 개선 가능

### 워크로드 특성화
- 연산율 및 유형, I/O 크기, 읽기/쓰기 비율
- 동기 쓰기 비율, 랜덤/순차 패턴
- 캐시 히트율/미스율

## 관측 도구

| 도구 | 용도 |
|------|------|
| `free -m` | 버퍼/페이지 캐시 크기 |
| `cachestat(8)` | 페이지 캐시 히트/미스율 실시간 |
| `ext4dist(8)` | ext4 연산 레이턴시 히스토그램 |
| `ext4slower(8)` | 임계값보다 느린 ext4 연산 |
| `opensnoop(8)` | open(2) 추적 |
| `filetop(8)` | 파일별 I/O 상위 목록 |
| `bpftrace` | 커스텀 VFS/FS 분석 |

### ext4dist 출력 해석
읽기에서 이중 분포: 0~15μs(캐시 히트) + 256~2048μs(디스크 읽기) → bimodal 패턴.

## 벤치마킹

### fio
비균등 랜덤 분포(pareto), 레이턴시 퍼센타일 보고 지원. 가장 권장.
```bash
fio --rw=randread --random_distribution=pareto:0.9 --bs=8k --size=5g --runtime=60
```

### Cache Flushing
```bash
echo 3 > /proc/sys/vm/drop_caches   # cold cache 벤치마크 시작점
```

**WSS가 메인 메모리보다 작으면 캐시를, WSS가 더 크면 디스크를 테스트하는 것이다.**

## 튜닝

### ext4
- `noatime`: atime 업데이트 비활성화 (I/O 감소)
- `relatime`: 기본값, 필요 시에만 atime 업데이트

### ZFS
| 파라미터 | 설명 |
|---------|------|
| `recordsize` | 파일 블록 크기 (기본 128KB). 소형 랜덤 I/O에는 작게 |
| `compression` | lz4 등 경량 압축으로 I/O 감소 = 성능 향상 가능 |
| `primarycache` | ARC 정책 (all/metadata/none) |

### 애플리케이션 레벨
- `posix_fadvise()`: SEQUENTIAL, RANDOM, DONTNEED 등 캐시 힌트
- `madvise()`: 메모리 매핑 힌트

## 관련 페이지

- [disk-performance](disk-performance.md) — 디스크 I/O
- [memory-performance](memory-performance.md) — 페이지 캐시
- [observability](observability.md) — 관측 도구 개요
- [bpf](../tools/bpf.md) — BPF 기반 FS 분석 도구
