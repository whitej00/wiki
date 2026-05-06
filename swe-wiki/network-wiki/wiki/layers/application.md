---
tags: [layer, application-layer]
layer: application
related: [protocol-layers, http, smtp, dns, cdn, tls, pgp]
source_count: 2
last_updated: 2026-04-09
---

# 애플리케이션 계층 (Application Layer)

인터넷 프로토콜 스택의 최상위 계층. 네트워크 애플리케이션과 그 프로토콜이 존재하는 곳이다.

## 역할

서로 다른 종단 시스템에서 실행되는 애플리케이션 프로세스 간의 통신을 정의한다. PDU(Protocol Data Unit) 명칭은 **메시지(Message)**.

## 애플리케이션 아키텍처

### 클라이언트-서버 (Client-Server)
- **서버**: 항상 켜져 있고 고정 IP 주소를 가진 호스트
- **클라이언트**: 서버에 요청을 보내는 호스트 (간헐적 연결, 동적 IP 가능)
- 클라이언트끼리는 직접 통신하지 않음
- 확장: 데이터 센터로 서버 역할 분산

### P2P (Peer-to-Peer)
- 전용 서버 최소/불필요
- 피어(peer)들이 직접 통신
- **자기 확장성(Self-scalability)**: 피어가 서비스를 소비하면서 동시에 제공
- 예: BitTorrent, DHT

## 소켓 (Socket)

프로세스와 네트워크 사이의 인터페이스 (**API**):
- 애플리케이션 계층과 전송 계층 사이의 "문(door)"
- 애플리케이션 개발자는 소켓의 애플리케이션 측만 제어
- 전송 프로토콜 선택 + 일부 파라미터 설정만 가능

## 전송 서비스 요구사항 (4차원)

| 차원 | 설명 | 예시 |
|---|---|---|
| 신뢰적 데이터 전송 | 데이터 손실 없이 전달 보장 | 이메일, 파일 전송 (손실 불가) |
| 처리량 | 최소 보장 처리량 | VoIP (32 kbps), 비디오 |
| 타이밍 | 지연 상한 보장 | 대화형 게임, 화상 회의 |
| 보안 | 암호화, 무결성, 인증 | 전자 상거래, 뱅킹 |

> 현재 인터넷의 TCP/UDP는 처리량과 타이밍 보장을 **제공하지 않는다**.

## 주요 프로토콜

| 프로토콜 | 용도 | 전송 프로토콜 | 포트 |
|---|---|---|---|
| [HTTP](../protocols/http.md) | 웹 | TCP | 80/443 |
| [SMTP](../protocols/smtp.md) | 이메일 전송 | TCP | 25 |
| [DNS](../protocols/dns.md) | 이름 해석 | UDP | 53 |
| FTP | 파일 전송 | TCP | 21 |
| IMAP | 이메일 접근 | TCP | 143 |
| DASH | 비디오 스트리밍 | TCP (HTTP) | 80/443 |

## 보안

애플리케이션 계층에서 보안을 제공하는 대표적 기술:
- [TLS](../protocols/tls.md) — TCP 위에서 기밀성, 무결성, 서버 인증 제공. 기술적으로 애플리케이션 계층에 위치하지만 전송 프로토콜처럼 동작
- [PGP](../protocols/pgp.md) — 이메일 보안 (기밀성 + 송신자 인증 + 무결성). Web of Trust 기반 공개키 인증

> 네트워크 계층([IPsec](../protocols/ipsec.md))이 "blanket coverage"를 제공하더라도, 사용자 수준 인증 등 상위 계층 보안이 별도로 필요하다. 또한 하위 계층 보안 배포를 기다리지 않고 애플리케이션 개발자가 직접 보안을 추가할 수 있다는 현실적 장점이 있다.

## 출처

- [kurose-ch2](../sources/kurose-ch2.md) — Kurose 8판 2.1절
- [kurose-ch8](../sources/kurose-ch8.md) — Kurose 8판 8.5, 8.6절
