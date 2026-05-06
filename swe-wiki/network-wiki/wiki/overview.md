---
tags: [overview]
last_updated: 2026-04-08
---

# 컴퓨터 네트워크 개관

컴퓨터 네트워크 전반을 다루는 위키입니다. OSI 7계층 및 TCP/IP 4계층 모델을 기반으로 프로토콜, 개념, 비교 분석을 체계적으로 정리합니다.

## 계층 모델

| TCP/IP 계층 | OSI 계층 | 역할 |
|---|---|---|
| Application | Application / Presentation / Session | 애플리케이션 간 데이터 교환 |
| Transport | Transport | 종단 간 신뢰성·흐름 제어 |
| Network | Network | 라우팅과 주소 지정 |
| Link | Data Link / Physical | 인접 노드 간 프레임 전달 |

## 인터넷 프로토콜 스택 (5계층)

Kurose 교재에서는 인터넷을 5개 계층으로 구분한다:

| 계층 | PDU | 역할 | 대표 프로토콜 |
|---|---|---|---|
| Application | 메시지 | 네트워크 애플리케이션 | HTTP, SMTP, DNS, FTP |
| Transport | 세그먼트 | 프로세스 간 통신 | [[TCP]], [[UDP]] |
| Network | 데이터그램 | 호스트 간 라우팅 | [[IP]] |
| Link | 프레임 | 인접 노드 간 전달 | 이더넷, WiFi |
| Physical | 비트 | 물리 매체 위 비트 전송 | 매체 종속적 |

자세한 내용은 [[protocol-layers]] 참조.

## 핵심 개념

- [[packet-switching]] — 패킷 교환: 인터넷의 데이터 전달 방식
- [[circuit-switching]] — 회선 교환: 전통적 전화망 방식
- [[encapsulation]] — 캡슐화: 계층 간 데이터 전달 메커니즘
- [[delay-loss-throughput]] — 지연, 손실, 처리량: 네트워크 성능 지표
- [[internet-structure]] — 인터넷 구조: ISP 계층과 상호 연결
