---
tags: [concept, network-layer, control-plane, SDN]
layer: network
related: [generalized-forwarding, routing-algorithms, OSPF, BGP]
source_count: 1
last_updated: 2026-04-08
---

# SDN 컨트롤 플레인 (SDN Control Plane)

**소프트웨어 정의 네트워킹(Software-Defined Networking, SDN)**의 컨트롤 플레인. 논리적으로 중앙집중화된 컨트롤러가 네트워크의 패킷 스위치들의 플로우 테이블을 계산·배포·관리한다. 전통적 per-router 컨트롤의 대안.

## 정의

SDN 컨트롤 플레인은 네트워크 전체의 포워딩을 제어하는 **네트워크 차원의 로직**으로, 데이터 플레인 스위치와 분리되어 별도의 서버에서 실행된다.

## SDN의 4대 특성

1. **Flow-based forwarding**: 목적지 IP뿐 아니라 전송·네트워크·링크 계층의 다양한 헤더 필드에 기반한 포워딩 (OpenFlow는 11개 필드 지원)
2. **Data/Control plane 분리**: 데이터 플레인은 단순한 스위치, 컨트롤 플레인은 별도 서버에서 실행
3. **외부 제어(Network control external to switches)**: 컨트롤 플레인이 스위치 외부의 소프트웨어로 구현
4. **프로그래머블 네트워크**: 컨트롤러 API를 통해 네트워크 동작을 프로그래밍

## SDN 컨트롤러 아키텍처

컨트롤러는 3개 계층으로 구성된다:

### 1. Communication Layer (Southbound Interface)
- SDN 컨트롤러 ↔ 스위치 간 통신
- **OpenFlow**, SNMP 등의 프로토콜 사용
- 스위치 이벤트(링크 up/down, 패킷 도착 등) 수신
- 플로우 테이블 엔트리 설치·수정

### 2. Network-wide State-Management Layer
- 네트워크 전체 상태 정보 유지: link-state, 호스트 정보, 스위치 정보, 플로우 테이블, 통계
- 분산 DB로 구현하여 일관성·가용성 확보

### 3. Application Layer (Northbound Interface)
- 네트워크 제어 애플리케이션과의 인터페이스
- **REST API**, Intent 등 제공
- 애플리케이션 예: 라우팅, 접근 제어, 부하 분산, 방화벽

> 컨트롤러는 **논리적으로** 중앙집중이지만, **물리적으로는** 분산 서버로 구현 (fault tolerance, scalability, high availability). 이벤트 순서, 일관성, 합의(consensus) 등 분산 시스템 문제가 발생한다.

## OpenFlow 프로토콜

SDN 컨트롤러와 스위치 간 통신 프로토콜. **TCP 포트 6653** 사용.

### Controller → Switch 메시지
| 메시지 | 기능 |
|---|---|
| **Configuration** | 스위치 설정 조회/변경 |
| **Modify-State** | 플로우 테이블 엔트리 추가/삭제/수정 |
| **Read-State** | 통계·카운터 수집 |
| **Send-Packet** | 특정 포트로 패킷 전송 |

### Switch → Controller 메시지
| 메시지 | 기능 |
|---|---|
| **Flow-Removed** | 플로우 엔트리 삭제 알림 |
| **Port-status** | 포트 상태 변경 알림 |
| **Packet-in** | 매칭 규칙 없는 패킷을 컨트롤러로 전달 |

## Data/Control Plane 상호작용 예시

링크 장애 시 SDN 동작 (s1-s2 링크 다운):
1. s1이 OpenFlow **port-status**로 컨트롤러에 링크 장애 통보
2. 컨트롤러가 link-state DB 업데이트, link-state 관리자에 통지
3. 라우팅 애플리케이션이 link-state 변경 알림 수신
4. 애플리케이션이 state-management 계층에서 상태 조회, 새 최단 경로 계산
5. 플로우 테이블 관리자가 갱신할 플로우 테이블 결정
6. OpenFlow로 영향받는 스위치(s1, s2, s4)의 플로우 테이블 갱신

## SDN 컨트롤러 사례

### OpenDaylight (ODL)
- Linux Foundation 오픈소스
- **Service Abstraction Layer (SAL)**: 컨트롤러의 핵심. 컴포넌트 간 통신, southbound 프로토콜 추상화
- Basic Network Functions: 토폴로지, 스위치 관리, 통계, 포워딩 규칙
- Northbound: REST/RESTCONF/NETCONF API
- Southbound: OpenFlow, NETCONF, SNMP, OVSDB
- **MD-SAL**: YANG 데이터 모델 기반 관리 (최신)

### ONOS
- Linux Foundation 오픈소스
- **Intent 프레임워크**: 고수준 서비스 요청 (예: "호스트 A↔B 연결 설정"). 구현 세부사항 추상화.
- **분산 코어**: ONOS를 여러 서버에 복제 배포. 서버 추가로 용량 확장.
- Northbound: REST API, Intent
- Southbound: OpenFlow, Netconf, OVSDB

### Google B4
- Google 데이터센터 간 WAN을 SDN으로 제어
- OpenFlow 기반, 맞춤 스위치 + Open Flow Agent (OFA)
- 링크 이용률 **70%+** 달성 (일반 WAN 대비 2~3배)
- 애플리케이션 우선순위별 대역폭 할당, 혼잡 시 대용량 데이터 복사가 양보
- BGP + IS-IS를 라우팅에, Paxos를 장애 복구에 사용

## SDN의 역사

| 시기 | 사건 |
|---|---|
| 2004 | data/control plane 분리 주장 (Feamster, Lakshman, RFC 3746) |
| 2007 | **Ethane 프로젝트**: flow-based 스위치 + 중앙 컨트롤러 (300대 스위치 운용) |
| 2008 | OpenFlow 프로젝트 시작, NOX 컨트롤러 |
| 2013 | Google B4 운용 |
| 현재 | NFV(Network Functions Virtualization)로 확장. ISP 대규모 도입 진행 중 |

## 출처
- [[kurose-ch5]]
