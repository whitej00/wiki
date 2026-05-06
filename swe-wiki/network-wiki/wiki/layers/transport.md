---
tags: [layer, transport]
layer: transport
related: [application, network, TCP, UDP, QUIC, multiplexing-demultiplexing]
source_count: 1
last_updated: 2026-04-08
---

# 전송 계층 (Transport Layer)

애플리케이션 계층과 네트워크 계층 사이에 위치하며, 서로 다른 호스트에서 실행되는 **애플리케이션 프로세스 간의 논리적 통신(Logical Communication)**을 제공하는 계층이다.

## 역할

네트워크 계층(IP)이 **호스트 간(host-to-host)** 통신을 제공하는 것에 반해, 전송 계층은 이를 **프로세스 간(process-to-process)** 통신으로 확장한다. 이를 위해 [[multiplexing-demultiplexing|다중화/역다중화]]를 수행한다.

전송 계층 프로토콜은 **종단 시스템(End System)**에서만 동작한다. 중간 라우터는 전송 계층 세그먼트의 필드를 검사하지 않으며, 네트워크 계층 데이터그램의 필드만 처리한다.

## 네트워크 계층과의 관계

전송 계층이 제공할 수 있는 서비스는 하위 네트워크 계층의 서비스 모델에 의해 제약된다. IP가 지연이나 대역폭을 보장하지 않으므로, 전송 계층도 이를 보장할 수 없다.

그러나 네트워크 계층이 제공하지 않는 서비스를 전송 계층이 독자적으로 제공할 수도 있다:
- IP가 비신뢰적이지만, TCP는 **신뢰적 데이터 전송**을 구현
- 네트워크 계층이 기밀성을 보장하지 않지만, 전송 계층에서 **암호화** 가능

## 인터넷 전송 프로토콜

| 프로토콜 | 특성 | 서비스 |
|----------|------|--------|
| [[TCP]] | 연결 지향, 신뢰적 | 신뢰적 전송, [[flow-control\|흐름 제어]], [[congestion-control\|혼잡 제어]] |
| [[UDP]] | 비연결, 비신뢰적 | 다중화/역다중화, 오류 검출(체크섬)만 제공 |
| [[QUIC]] | UDP 기반 애플리케이션 계층 프로토콜 | 연결 지향, 암호화, 스트림 멀티플렉싱, 혼잡 제어 |

TCP와 UDP 모두 제공하는 최소 서비스:
1. **프로세스-프로세스 데이터 전달** ([[multiplexing-demultiplexing]])
2. **오류 검출** (체크섬)

TCP가 추가로 제공하는 서비스:
1. **[[reliable-data-transfer|신뢰적 데이터 전송]]** — 순서번호, 확인응답, 타이머, 재전송
2. **[[flow-control|흐름 제어]]** — 수신자 버퍼 오버플로 방지
3. **[[congestion-control|혼잡 제어]]** — 네트워크 혼잡 방지 (인터넷 전체의 이익)

## 용어

- **세그먼트(Segment)**: 전송 계층 패킷의 일반적 명칭. TCP에서는 segment, UDP에서는 datagram이라고도 하나 본 위키에서는 둘 다 segment로 통칭
- **데이터그램(Datagram)**: 네트워크 계층 패킷 (IP datagram)

## 핵심 개념

- [[multiplexing-demultiplexing]] — 프로세스 간 데이터 전달의 기본 메커니즘
- [[reliable-data-transfer]] — 비신뢰 채널 위에서 신뢰적 전송을 구축하는 원리
- [[flow-control]] — 송신자-수신자 간 속도 매칭
- [[congestion-control]] — 네트워크 혼잡 감지 및 대응
- [[end-to-end-principle]] — 종단간 설계 원칙

## 포함 프로토콜

- [[TCP]] — 연결 지향 신뢰적 전송 프로토콜
- [[UDP]] — 비연결형 최소 기능 전송 프로토콜
- [[QUIC]] — UDP 위에 구축된 차세대 전송 기능 프로토콜

## 출처
- [[kurose-ch3]]
