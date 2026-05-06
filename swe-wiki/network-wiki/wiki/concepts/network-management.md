---
tags: [concept, network-layer, management]
layer: network
related: [OSPF, sdn-control-plane, ICMP]
source_count: 1
last_updated: 2026-04-08
---

# 네트워크 관리 (Network Management)

네트워크의 하드웨어, 소프트웨어, 인간 요소를 배치·통합·조정하여 네트워크를 모니터링, 테스트, 설정, 분석, 평가, 제어하는 활동. 실시간으로 네트워크 자원의 운영 성능과 QoS 요구사항을 합리적 비용으로 충족시키는 것이 목표다.

## 관리 프레임워크

네트워크 관리 시스템의 핵심 구성 요소:

### 1. Managing Server (관리 서버)
- NOC(Network Operations Center)에서 운영
- 관리 정보 수집·처리·분석, 명령 발송
- 실무에서는 복수 서버 운용

### 2. Managed Device (관리 대상 장치)
- 호스트, 라우터, 스위치, 미들박스, 모뎀 등
- 각 장치에는 여러 **관리 가능 컴포넌트**(네트워크 인터페이스, 라우팅 프로토콜 설정 등)가 있음

### 3. Data (데이터)
| 종류 | 설명 | 예시 |
|---|---|---|
| **Configuration data** | 관리자가 설정한 정보 | IP 주소, 인터페이스 속도 |
| **Operational data** | 장치가 운영 중 수집한 정보 | OSPF 이웃 목록 |
| **Device statistics** | 상태 카운터 | 드롭된 패킷 수, 팬 속도 |

### 4. Network Management Agent
- 관리 대상 장치에서 실행되는 소프트웨어 프로세스
- 관리 서버의 명령을 로컬에서 수행
- SDN의 Control Agent(CA)와 유사한 역할

### 5. Network Management Protocol
- 관리 서버 ↔ 에이전트 간 통신
- 네트워크를 직접 관리하는 것이 아니라 관리할 수 있는 **능력을 제공**

## 3가지 관리 접근법

### 1. CLI (Command Line Interface)
- 관리자가 장치에 직접 명령 입력 (콘솔, Telnet, SSH)
- 장점: 직관적, 유연
- 단점: 벤더별 상이, 오류 발생 쉬움, 대규모 네트워크에서 확장 어려움

### 2. SNMP / MIB
- 장치의 MIB 객체를 SNMP로 조회/설정
- 주로 **모니터링**에 사용 (SetRequest는 보안 문제로 거의 사용 안 함)
- 장치를 **개별적으로** 관리

### 3. NETCONF / YANG
- 네트워크 **전체**를 추상적·통합적으로 관리
- **설정 관리**에 강점: 원자적 트랜잭션, 다중 장치 동시 변경
- SDN과 함께 발전

## SNMP (Simple Network Management Protocol)

### 개요
- 버전: SNMPv3 [RFC 3410]
- 전송: 주로 **UDP** (RFC 3417: "preferred transport mapping")
- 애플리케이션 계층 프로토콜
- 요청-응답 모델 + 비동기 trap

### PDU 타입 (7종)

| PDU | 방향 | 설명 |
|---|---|---|
| **GetRequest** | manager→agent | MIB 객체 값 조회 |
| **GetNextRequest** | manager→agent | MIB 리스트/테이블 순차 조회 |
| **GetBulkRequest** | manager→agent | 대량 데이터 일괄 조회 |
| **SetRequest** | manager→agent | MIB 객체 값 설정 |
| **InformRequest** | manager→manager | 원격 관리 서버에 MIB 정보 통보 |
| **Response** | agent→manager | 요청에 대한 응답 |
| **SNMPv2-Trap** | agent→manager | 비동기 이벤트 알림 (cold start, 링크 up/down, 이웃 상실, 인증 실패 등) |

### 신뢰성
- UDP 사용으로 요청/응답 유실 가능
- Request ID로 매칭하여 유실 감지
- 재전송 정책은 관리 서버의 재량 (표준에서 강제하지 않음)

### 보안 (SNMPv3)
- SNMPv3에서 보안·관리 기능 대폭 강화
- 보안 부재로 인해 SNMP는 모니터링 중심으로 사용됨 (SetRequest 거의 미사용)

## MIB (Management Information Base)

- 관리 대상 장치의 운영 상태·설정 데이터를 **객체**로 표현
- **SMI** (Structure of Management Information)라는 데이터 정의 언어로 명세
- **MIB 모듈**: 관련 객체의 묶음. 400+ MIB 모듈이 IETF RFC로 정의
- 예시: ipSystemStatsInDelivers (RFC 4293) — 상위 프로토콜에 전달된 IP 데이터그램 수를 추적하는 32비트 읽기전용 카운터

## NETCONF (Network Configuration Protocol)

### 개요
- RFC 6241
- **RPC 패러다임**: XML 인코딩 메시지를 **TLS/TCP** 보안 세션 위에서 교환
- 관리 서버가 "client", 관리 장치가 "server" (NETCONF 용어)

### 세션 흐름
1. 보안 연결 수립
2. `<hello>` 교환으로 capabilities 협상
3. `<rpc>` / `<rpc-reply>` 쌍으로 설정 조회·변경
4. 장치가 `<notification>`으로 비동기 이벤트 전송
5. `<close-session>`으로 종료

### 주요 오퍼레이션

| 오퍼레이션 | 설명 |
|---|---|
| `<get-config>` | 설정 조회 (running config 포함) |
| `<get>` | 설정 + 운영 상태 데이터 조회 |
| `<edit-config>` | 설정 변경 (성공 시 `<ok>`, 실패 시 롤백 가능) |
| `<lock>` / `<unlock>` | 설정 데이터스토어 잠금/해제 (다른 소스의 동시 변경 방지) |
| `<create-subscription>` / `<notification>` | 이벤트 구독·알림 |

### SNMP 대비 장점
- **원자적 트랜잭션**: 다수 장치에 대한 설정을 그룹으로 적용하고, 실패 시 전체 롤백
- **설정 중심**: 네트워크 전체의 **configuration**에 집중 (개별 장치 모니터링이 아닌)
- XML 기반으로 사람이 읽을 수 있음

## YANG

- RFC 6020
- NETCONF에서 사용하는 **데이터 모델링 언어**
- SNMP의 SMI와 유사한 역할
- 모듈 단위로 정의, XML 문서 자동 생성 가능
- **정확성·일관성 제약(constraints)** 표현 가능 — NETCONF 설정의 유효성 검증에 활용
- 알림(notification) 명세에도 사용

## 출처
- [[kurose-ch5]]
