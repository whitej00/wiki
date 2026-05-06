---
title: "Systems Performance: Enterprise and the Cloud, 2nd Edition"
type: source
tags: [systems-performance, linux, observability, brendan-gregg]
created: 2026-04-05
updated: 2026-04-05
sources: [Systems.Performance.Enterprise.and.the.Cloud.2nd.Edition.2020.12.pdf]
---

# Systems Performance: Enterprise and the Cloud, 2nd Edition

## 기본 정보

- **저자:** Brendan Gregg
- **출판:** 2020년 12월, Addison-Wesley (Pearson)
- **분량:** 929페이지, 16개 챕터 + 부록
- **ISBN:** 978-0-13-682015-4

## 개요

시스템 성능 분석의 바이블. 스토리지부터 애플리케이션 소프트웨어까지 데이터 경로 상의 모든 구성 요소를 아우르는 "full stack" 성능 분석을 다룬다. 1판(2013) 대비 BPF/eBPF, 클라우드 컴퓨팅(컨테이너, Kubernetes), 최신 Linux 커널 기능이 대폭 추가되었다.

## 목차 및 관련 위키 페이지

### Part I: 기초 (필수 배경지식)

| 챕터 | 주제 | 위키 페이지 |
|------|------|------------|
| Ch1 | Introduction | [[systems-performance-methodology]] |
| Ch2 | Methodologies | [[systems-performance-methodology]], [[use-method]] |
| Ch3 | Operating Systems | [[os-kernel]] |
| Ch4 | Observability Tools | [[observability]] |

### Part II: 리소스별 분석

| 챕터 | 주제 | 위키 페이지 |
|------|------|------------|
| Ch5 | Applications | [[application-performance]] |
| Ch6 | CPUs | [[cpu-performance]] |
| Ch7 | Memory | [[memory-performance]] |
| Ch8 | File Systems | [[filesystem-performance]] |
| Ch9 | Disks | [[disk-performance]] |
| Ch10 | Network | [[network-performance]] |
| Ch11 | Cloud Computing | [[cloud-performance]] |
| Ch12 | Benchmarking | [[benchmarking]] |

### Part III: 고급 도구

| 챕터 | 주제 | 위키 페이지 |
|------|------|------------|
| Ch13 | perf | [[perf]] |
| Ch14 | Ftrace | [[ftrace]] |
| Ch15 | BPF | [[bpf]] |
| Ch16 | Case Study | (Netflix 사례) |

## 핵심 메시지

1. **성능 분석은 두 손이 필요하다** — 관찰(observability)과 실험(experimentation)을 함께 사용
2. **최고의 성능 향상은 불필요한 작업을 제거하는 것** — Performance Mantras 참조
3. **USE 방법론**으로 모든 리소스의 Utilization, Saturation, Errors를 체계적으로 점검
4. **BPF/eBPF**가 프로덕션 환경에서의 성능 관찰 방식을 혁신적으로 변화시킴
5. **레이턴시**가 가장 강력한 성능 메트릭 — 최대 속도 향상을 정량적으로 계산 가능

## 저자 소개

Brendan Gregg는 Netflix의 시니어 성능 아키텍트. DTrace, BPF 기반 성능 도구, Flame Graph를 발명/대중화한 시스템 성능 분야의 세계적 권위자다.
