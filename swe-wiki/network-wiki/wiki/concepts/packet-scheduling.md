---
tags: [concept, network-layer]
layer: network
related: [router-architecture, delay-loss-throughput]
source_count: 1
last_updated: 2026-04-08
---

# 패킷 스케줄링 (Packet Scheduling)

라우터 출력 포트에서 큐에 대기 중인 패킷들 중 **어떤 패킷을 다음에 전송할지** 결정하는 알고리즘이다.

## FIFO (First-In-First-Out)

**FCFS(First-Come-First-Served)**라고도 한다. 도착 순서대로 전송하는 가장 단순한 방식.

- 버퍼가 가득 차면 패킷 폐기 정책 적용 (drop-tail 등)
- 패킷 간 차별화 없음

## Priority Queuing (우선순위 큐잉)

패킷을 **우선순위 클래스**로 분류하여 별도의 큐에 저장. 높은 우선순위 큐가 비어있지 않은 한 항상 높은 클래스를 먼저 서비스한다.

- 분류 기준: 출발지/목적지 포트 번호, IP 주소 등 헤더 필드
- 같은 클래스 내에서는 FIFO
- **Non-preemptive**: 현재 전송 중인 패킷은 중단하지 않음

예: 네트워크 관리 패킷(SNMP) > 실시간 음성 > 일반 이메일

## Round Robin (RR, 라운드 로빈)

패킷을 **클래스로 분류**하되, 클래스 간에 **교대로** 서비스한다.

- 클래스 1 패킷 1개 → 클래스 2 패킷 1개 → 클래스 1 패킷 1개 → ...
- **Work-conserving**: 한 클래스의 큐가 비면 다음 클래스로 즉시 이동
- 우선순위 큐잉과 달리 어떤 클래스도 완전히 차단(starvation)되지 않음

## WFQ (Weighted Fair Queuing, 가중 공정 큐잉)

Round Robin에 **가중치**를 부여한 일반화된 형태. 가장 널리 구현된 스케줄링 알고리즘.

- 각 클래스 i에 가중치 w_i 할당
- 라운드 로빈 방식으로 서비스하되, 각 클래스가 받는 서비스 비율이 가중치에 비례
- 모든 클래스가 큐에 패킷을 가지고 있을 때, 클래스 i의 **보장 처리량**:

  ```
  R · w_i / Σw_j    (R = 링크 전송률)
  ```

- Work-conserving: 다른 클래스가 비면 남은 대역폭을 자동으로 흡수
- WFQ도 non-preemptive

## 망 중립성 (Net Neutrality)

패킷 스케줄링과 직접 관련된 정책 이슈.

2015년 FCC의 3대 원칙:
1. **No Blocking**: 합법 콘텐츠 차단 금지
2. **No Throttling**: 합법 트래픽 속도 저하 금지
3. **No Paid Prioritization**: 유료 우선 처리 금지

2017년 FCC가 이 규칙을 완화. 논쟁은 현재 진행 중.

## 출처
- [[kurose-ch4]]
