---
title: "벤치마킹"
type: concept
tags: [benchmarking, performance, testing, methodology]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# 벤치마킹

## 왜 벤치마킹이 어려운가

> "벤치마크를 실행하는 주니어 직원과 결과를 설명하는 성능 전문가를 분리하지 말 것. 전문가가 실행 중에 분석해야 한다."

## 벤치마킹 실패 16가지

| # | 실패 유형 | 설명 |
|---|----------|------|
| 1 | Casual Benchmarking | A를 벤치마크하지만 실제로 B를 측정하고 C를 결론 |
| 2 | Blind Faith | 인기 있는 도구가 정확하다는 맹신 |
| 3 | Numbers Without Analysis | 분석 없이 숫자만 제시 |
| 4 | Complex Benchmark Tools | 도구 자체가 병목 (단일 스레드 등) |
| 5 | Testing the Wrong Thing | 캐시에서 동작하는데 디스크 벤치마크 |
| 6 | Ignoring the Environment | 테스트 환경 ≠ 프로덕션 |
| 7 | Ignoring Errors | 모든 요청이 에러 → 타임아웃이 결과 |
| 8 | Ignoring Variance | 평균만 시뮬레이션, 버스트 무시 |
| 9 | Ignoring Perturbations | 백업, 모니터링 에이전트, 이웃 테넌트 |
| 10 | Changing Multiple Factors | 통제 안 된 변수 |
| 11 | Benchmark Paradox | 성공 확률 50%인 벤치마크 3개 모두 이길 확률은 12.5% |
| 12 | Benchmarking the Competition | 경쟁 제품 튜닝 없이 비교 |
| 13 | Friendly Fire | 자사 제품 최고 설정 미적용 |
| 14 | Misleading Benchmarks | 기술적 정확하나 중요 정보 생략 |
| 15 | Benchmark Specials | 벤치마크만을 위한 최적화 |
| 16 | Cheating | 허위 결과 |

## 벤치마킹 유형

### Micro-Benchmarking
단일 연산 유형 테스트. 단순, 반복 가능, 빠른 테스트.

도구: SysBench(CPU), fio(파일시스템), iperf(네트워크), lmbench(메모리)

### Simulation (매크로벤치마크)
실제 워크로드 시뮬레이션. wrk, siege 등 HTTP 부하 테스트.

### Replay
실제 추적 로그 재생. 캐시 크기 차이 등으로 부정확할 수 있음.

### Industry Standards
TPC-C(OLTP), SPEC CPU 2017 등 독립 기관 표준.

## Active vs Passive Benchmarking

### Passive (안티패턴)
도구 실행 → 결과 수집 → 슬라이드 작성. **분석 없음.**

### Active (권장)
실행 **중에** 성능 분석:
- 벤치마크가 실제 무엇을 테스트하는지 확인
- 한계 요인 식별 (시스템 or 도구 자체?)
- Flame graph로 CPU 사용 분석

**bonnie++ 사례**: "하드 드라이브 테스트"라고 했지만 실제로는 파일시스템 캐시만 테스트하고 있었음 → `cachestat`과 `bpftrace`로 확인.

## Ramping Load

부하를 점진적으로 증가시켜 최대 처리량 파악. **"수용 가능한 레이턴시에서의 최대 IOPS"**가 진짜 결과.

## Sanity Check

결과가 알려진 물리 한계를 초과하는지 계산:
```
NFS 서버 1GbE 포트 → 50,000 IOPS × 8KB = 3.2Gbps > 1Gbps 한계 → 결과 무효
```

## 벤치마킹 체크리스트 (6가지 핵심 질문)

1. **Why not double?** — 2배가 아닌 이유는? (= 한계 요인이 무엇인가)
2. **Did it break limits?** — 물리 한계를 초과했는가
3. **Did it error?** — 에러가 결과를 왜곡하고 있지 않은가
4. **Does it reproduce?** — 재현 가능한가? 표준 편차는?
5. **Does it matter?** — 프로덕션 워크로드와 관련 있는가
6. **Did it even happen?** — 트래픽이 실제 대상에 도달했는가

## 관련 페이지

- [systems-performance-methodology](systems-performance-methodology.md) — 분석 방법론
- [cpu-performance](cpu-performance.md) — CPU 벤치마킹
- [disk-performance](disk-performance.md) — 디스크 벤치마킹
- [network-performance](network-performance.md) — 네트워크 벤치마킹
