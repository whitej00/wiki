---
title: "시스템 성능 분석 방법론"
type: concept
tags: [performance, methodology, latency, scalability, statistics]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# 시스템 성능 분석 방법론

시스템 성능(Systems Performance)은 스토리지부터 애플리케이션까지 **데이터 경로 상의 모든 구성 요소**를 포함하는 전체 시스템의 성능을 분석하는 분야다.

## 왜 성능 분석이 어려운가

1. **주관성**: "1ms 디스크 응답이 좋은가?"는 맥락에 따라 다르다 → 명확한 목표치(SLO) 설정 필요
2. **복잡성**: 단독으로 정상인 서브시스템들이 상호작용하여 문제 발생 (캐스케이딩 실패)
3. **다중 원인**: 단일 근본 원인이 아닌 여러 기여 요소가 동시 작용
4. **다중 이슈**: 진짜 과제는 이슈를 찾는 것이 아니라 **가장 중요한 이슈를 정량화**하는 것

## 핵심 용어

| 용어 | 정의 |
|------|------|
| **Latency** | 작업 완료까지의 대기 시간. 가장 강력한 성능 메트릭 |
| **IOPS** | 초당 I/O 작업 수 |
| **Throughput** | 초당 처리된 작업량 (바이트 또는 작업 수) |
| **Utilization** | 리소스가 작업을 수행하는 시간 비율. `U = B/T` |
| **Saturation** | 리소스가 처리하지 못해 대기 중인 작업의 정도 |
| **Bottleneck** | 시스템 성능을 제한하는 리소스 |

## 레이턴시 — 가장 강력한 메트릭

레이턴시로 **최대 예상 속도 향상(speedup)**을 계산할 수 있다:

```
예시: DB 쿼리 100ms 중 디스크 읽기 대기 80ms
→ 캐싱으로 디스크 제거 시: 100ms → 20ms = 5x 향상
```

IOPS 같은 다른 메트릭으로는 이런 계산이 불가능하다.

### 시간 규모 (Time Scales)

3.5GHz 프로세서에서 1 CPU 사이클 = 1초라면:

| 이벤트 | 실제 | 스케일 |
|--------|------|--------|
| L1 캐시 | 0.9 ns | 3초 |
| L2 캐시 | 3 ns | 10초 |
| L3 캐시 | 10 ns | 33초 |
| DRAM | 100 ns | 6분 |
| SSD I/O | 10–100 μs | 9–90시간 |
| HDD I/O | 1–10 ms | 1–12개월 |
| SF→뉴욕 | 40 ms | 4년 |

## 분석 관점

### Resource Analysis (리소스 분석)
- **방향**: 아래(하드웨어)에서 위(애플리케이션)로
- **집중**: IOPS, Throughput, Utilization, Saturation
- **도구**: vmstat, iostat, mpstat

### Workload Analysis (워크로드 분석)
- **방향**: 위(애플리케이션)에서 아래(하드웨어)로
- **집중**: 요청 속도, 레이턴시, 완료/오류
- **원칙**: "가장 빠른 쿼리는 아예 하지 않는 쿼리다"

## 주요 방법론

### Anti-Methods (피해야 할 것들)

| 안티-방법론 | 설명 |
|------------|------|
| **Streetlight** | 아는 도구만 사용, 밝은 곳만 찾기 |
| **Random Change** | 임의로 변경하고 개선 여부 확인 |
| **Blame-Someone-Else** | 데이터 없이 다른 팀 탓하기 |

### Problem Statement (문제 진술)

새 이슈 접수 시 **가장 먼저** 물어볼 6가지:

1. 성능 문제가 있다고 생각하는 이유?
2. 이 시스템이 제대로 수행한 적이 있는가?
3. 최근 변경된 사항? (소프트웨어, 하드웨어, 부하)
4. 레이턴시 또는 런타임으로 문제를 표현할 수 있는가?
5. 다른 사람/애플리케이션도 영향을 받는가?
6. 환경은? (OS, 하드웨어, 버전, 구성)

### Scientific Method (과학적 방법)

```
질문 → 가설 → 예측 → 테스트 → 분석 → (반복)
```

실험적 테스트에서 변경이 해결하지 못하면 **반드시 되돌린 후** 다른 가설 시도.

### USE Method

→ [use-method](use-method.md) 참조. 모든 리소스에 대해 Utilization, Saturation, Errors 확인.

### RED Method

클라우드 마이크로서비스용. 모든 **서비스**에 대해:
- **Request rate**: 초당 요청 수
- **Errors**: 실패한 요청 수
- **Duration**: 요청 완료 시간 (퍼센타일 포함)

USE는 머신 건강, RED는 사용자 건강.

### Drill-Down Analysis (드릴다운 분석)

```
Monitoring → Identification → Analysis
```

**Five Whys**: "왜?"를 5번 반복하여 근본 원인 도달.

### Latency Analysis (레이턴시 분석)

작업 시간을 두 부분으로 나누고 더 큰 부분을 다시 세분화 — **레이턴시의 이진 탐색**.

```
MySQL 쿼리 느림 → on-CPU? off-CPU? → off-CPU → 파일시스템 I/O?
→ 디스크 I/O? 잠금? → 디스크 → 큐잉? 서비싱? → ...
```

### Performance Mantras (성능 만트라)

효과 순서대로:
1. **하지 마라** — 불필요한 작업 제거
2. **다시 하지 마라** — 캐싱
3. **덜 해라** — 빈도 감소
4. **나중에 해라** — write-back 캐싱
5. **몰래 해라** — 비업무 시간 스케줄링
6. **동시에 해라** — 멀티스레딩
7. **더 싸게 해라** — 더 빠른 하드웨어

## 모델링

### Amdahl's Law

```
C(N) = N / (1 + α(N – 1))
```
- α = 직렬성 정도 (0=완전 병렬, 1=완전 직렬)

### Universal Scalability Law (USL)

```
C(N) = N / (1 + α(N – 1) + βN(N – 1))
```
- β = 일관성(coherency) 파라미터. β > 0이면 피크 이후 성능 감소.

### Queueing Theory

**Little's Law**: `L = λW` (시스템 내 평균 요청 수 = 도착률 × 평균 시간)

M/D/1 모델에서 서비스 시간 1ms 기준:
- **60% 활용률** → 응답 시간 **2배**
- **80% 활용률** → 응답 시간 **3배**

이것이 디스크 활용률 60%부터 문제가 되는 이유.

## 캐싱

```
hit ratio = hits / (hits + misses)
runtime = (hit_rate × hit_latency) + (miss_rate × miss_latency)
```

히트율 98%→99% 향상이 10%→11%보다 **훨씬 큰** 성능 향상. 두 계층의 속도 차이가 클수록 이 비선형성은 더 급격해진다.

캐시 워밍업 예시: 128GB DRAM + 600GB 플래시 + HDD에서 8KB I/O, 2,000 reads/s 디스크 성능 → DRAM 워밍업 **2시간+**, 플래시 워밍업 **10시간+**.

## 통계

- **평균의 함정**: 캐시 히트(1ms)와 미스(7ms)의 평균 3.3ms는 어디도 대표하지 않는다
- **퍼센타일**: 99th percentile로 가장 느린 1% 정량화
- **다중모달 분포**: 시스템 성능은 종종 bimodal (캐시 히트 vs 미스)
- **히트맵**: 산포도의 확장성 문제를 해결, 수백만 이벤트를 시각화 가능

## 시각화

| 유형 | 용도 |
|------|------|
| **Line Chart** | 시간 경과에 따른 메트릭 변화 |
| **Scatter Plot** | 개별 이벤트 시각화, 아웃라이어 발견 |
| **Heat Map** | 대규모 이벤트의 분포 패턴 (Brendan Gregg 발명, 2008) |
| **Flame Graph** | CPU 프로파일 시각화, 코드 경로별 CPU 시간 |

## 튜닝 노력의 효과

작업이 수행되는 곳에 **가까울수록** 효과적:
- 애플리케이션 레이어: 불필요한 쿼리 제거 → **20x** 가능
- 스토리지 레이어 튜닝: 이미 상위 코드 실행 후 → **~20%** 수준

## Known-Unknowns

- **Known-knowns**: CPU 활용률이 10%임을 안다
- **Known-unknowns**: 프로파일링 가능하지만 아직 하지 않았다
- **Unknown-unknowns**: 디바이스 인터럽트의 CPU 영향을 모른다는 것조차 모른다

→ 많이 배울수록 unknown-unknowns가 known-unknowns로 변한다.

## 관련 페이지

- [use-method](use-method.md) — USE 방법론 상세
- [observability](observability.md) — 관측 도구 개요
- [cpu-performance](cpu-performance.md) — CPU 성능 분석
- [memory-performance](memory-performance.md) — 메모리 성능 분석
- [benchmarking](benchmarking.md) — 벤치마킹 방법론
