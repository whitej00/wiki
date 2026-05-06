---
tags: [protocol, network-layer, control-plane, inter-AS]
layer: network
rfc: 4271
related: [OSPF, routing-algorithms, ip-addressing, internet-structure, sdn-control-plane]
source_count: 1
last_updated: 2026-04-08
---

# BGP (Border Gateway Protocol)

인터넷의 **inter-AS(자율 시스템 간) 라우팅 프로토콜**. 수천 개의 ISP를 하나의 인터넷으로 연결하는 "접착제" 역할을 하며, IP와 함께 인터넷에서 가장 중요한 프로토콜이다.

## 한 줄 요약

AS 간 경로 도달성 정보를 교환하고 정책 기반으로 최적 경로를 선택하는 분산형 비동기 라우팅 프로토콜.

## BGP의 역할

BGP는 각 라우터에 두 가지 수단을 제공한다:

1. **이웃 AS로부터 프리픽스 도달성 정보 획득**: 서브넷이 자신의 존재를 인터넷 전체에 광고
2. **프리픽스까지의 "최적" 경로 결정**: 도달성 정보와 정책을 기반으로 경로 선택

BGP에서 목적지는 개별 IP가 아닌 **CIDRized 프리픽스** 단위 (예: 138.16.68/22).

## BGP 세션

라우터 쌍이 **TCP 포트 179**로 세미영구 연결을 맺어 BGP 메시지를 교환한다.

| 세션 유형 | 설명 |
|---|---|
| **eBGP** (external BGP) | 서로 다른 AS의 게이트웨이 라우터 간 연결 |
| **iBGP** (internal BGP) | 같은 AS 내 라우터 간 연결. 물리 링크와 1:1 대응 아님 (풀 메시 구성 일반적) |

도달성 정보 전파 순서: AS3 → eBGP → AS2 게이트웨이 → iBGP → AS2 내부 → eBGP → AS1 게이트웨이 → iBGP → AS1 내부

## BGP 속성 (BGP Attributes)

프리픽스 광고 시 여러 **속성(attribute)**이 함께 전달된다. 프리픽스 + 속성 = **BGP route**.

### AS-PATH
- 광고가 경유한 AS의 목록 (예: "AS2 AS3 x")
- AS가 프리픽스를 다음 AS에 전달할 때 자신의 ASN을 추가
- **루프 탐지**: 자신의 AS가 AS-PATH에 이미 있으면 광고 폐기

### NEXT-HOP
- AS-PATH가 시작되는 **라우터 인터페이스의 IP 주소**
- 반드시 자기 AS에 속한 주소는 아님 (다른 AS 라우터의 인터페이스)
- inter-AS와 intra-AS 라우팅을 연결하는 핵심 링크

## 경로 선택 알고리즘 (Route-Selection Algorithm)

동일 프리픽스에 대해 여러 경로가 존재할 때, 다음 규칙을 **순서대로** 적용:

1. **최고 local preference**: AS 정책에 의해 설정되는 값. 관리자 재량.
2. **최단 AS-PATH**: AS 홉 수가 가장 적은 경로. (이 규칙만 사용 시 DV 알고리즘과 유사)
3. **Hot potato routing**: 남은 경로 중 NEXT-HOP까지의 intra-AS 비용이 가장 작은 경로. 자기 AS에서 패킷을 최대한 빨리 내보내는 "이기적" 전략.
4. **BGP identifier**: 위 규칙으로도 구분 불가 시, BGP 라우터 식별자로 타이브레이크.

> local preference가 최우선이므로, BGP는 단순한 최단경로 알고리즘이 아니라 **정책 우선** 프로토콜이다.

## Hot Potato Routing

패킷을 자기 AS 밖으로 **최소 비용으로 가능한 빨리** 넘기는 라우팅. 종단간 경로의 나머지 부분(다른 AS 영역)의 비용은 고려하지 않는다. 같은 AS 내의 라우터라도 같은 프리픽스에 대해 서로 다른 AS 경로를 선택할 수 있다.

## IP-Anycast

BGP를 활용한 **IP-Anycast** 서비스:
1. 여러 지리적 위치의 서버에 **동일한 IP 주소**를 할당
2. 각 서버에서 해당 IP를 BGP로 광고
3. BGP route-selection이 각 라우터에서 가장 "가까운" 서버로 라우팅

**주요 활용**: DNS 루트 서버 (13개 IP 주소, 100+ 물리 서버). CDN에서는 TCP 연결이 다른 서버로 전환될 수 있어 실제로는 DNS 기반 리다이렉션을 선호.

## 라우팅 정책 (Routing Policy)

ISP 간 상업적 관계에 따라 BGP 경로 광고를 제어:

### Provider / Customer / Peer 관계
- **Customer**: provider에게 비용을 내고 트래픽 전달을 받음
- **Provider**: customer의 트래픽을 인터넷에 전달
- **Peer**: 상호 무료로 고객 트래픽을 교환

### 정책 구현
- **Access ISP** (stub network): 자기 네트워크 프리픽스만 광고. 다른 AS의 트래픽을 중계하지 않음.
- **Multi-homed access ISP** (예: X가 B, C에 연결): B, C에게 자기 프리픽스만 광고. B↔C 간 트랜짓 트래픽이 X를 경유하지 않도록 방지.
- **Backbone ISP**: 자기 네트워크를 경유하는 트래픽의 출발지 또는 목적지가 자기 고객이어야 함. 그렇지 않으면 "무임승차".

### inter-AS vs intra-AS 라우팅이 다른 이유

| | inter-AS (BGP) | intra-AS (OSPF) |
|---|---|---|
| **정책** | 최우선. 경로 광고/선택 모두 정책에 지배됨 | 정책 덜 중요. 같은 관리 하이므로 |
| **규모** | 대규모. AS 수가 커도 AS-PATH 기준이라 확장 가능 | AS가 커지면 분할하여 inter-AS 라우팅 사용 |
| **성능** | 이차적. 정책이 성능보다 우선 | 중요. 비용 최적 경로 추구 |

## 인터넷 프레즌스 획득 (Putting the Pieces Together)

회사가 인터넷에 연결되는 과정:
1. ISP 계약 → 게이트웨이 라우터 연결 → IP 주소 범위 할당 (예: /24)
2. 도메인 등록 → DNS 서버 설정 → DNS 레코드 등록
3. ISP가 회사 프리픽스를 **BGP로 광고** → 전 세계 라우터가 해당 프리픽스 인지
4. 외부 사용자의 데이터그램이 BGP 경로를 따라 회사 서버에 도달

## 출처
- [[kurose-ch5]]
