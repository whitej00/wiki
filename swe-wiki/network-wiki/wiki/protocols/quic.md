---
tags: [protocol, application-layer, transport-layer]
layer: application
rfc: QUIC 2020 (draft)
related: [TCP, UDP, HTTP, congestion-control, reliable-data-transfer]
source_count: 1
last_updated: 2026-04-08
---

# QUIC (Quick UDP Internet Connections)

[[UDP]] 위에 구축된 **애플리케이션 계층 프로토콜**로, [[TCP]]의 신뢰적 전송, [[congestion-control|혼잡 제어]], 연결 설정 기능과 TLS 암호화를 통합하여 전송 계층 서비스의 성능을 개선한다. HTTP/3의 기반 프로토콜이다.

## 등장 배경

TCP와 UDP 어느 쪽의 서비스 모델도 완벽하게 맞지 않는 애플리케이션이 존재한다. TCP의 특정 기능(혼잡 제어, 신뢰적 전송)은 필요하지만 다른 측면(HOL 블로킹, 긴 연결 설정)은 불리한 경우, 애플리케이션 설계자는 UDP 위에 필요한 기능을 직접 구현하는 방식을 택할 수 있다. QUIC가 바로 이 접근을 체계화한 프로토콜이다.

QUIC는 Google이 개발하여 자사 공개 웹 서버, YouTube, Chrome 브라우저, Android Google 검색 앱 등에 광범위하게 배포했다. 인터넷 트래픽의 7% 이상이 QUIC를 사용한다.

## 프로토콜 스택 비교

**전통적 HTTP/1.1-2**:
```
Application:  HTTP/2
              TLS
Transport:    TCP
Network:      IP
```

**QUIC 기반 HTTP/3**:
```
Application:  HTTP/2 (slimmed) = HTTP/3
              QUIC
Transport:    UDP
Network:      IP
```

## 주요 특성

### 1. 연결 지향 + 보안

QUIC는 TCP처럼 연결 지향 프로토콜이며, 두 엔드포인트 간 핸드셰이크가 필요하다. 연결 상태에는 **연결 ID(Connection ID)**와 **목적지 연결 ID**가 포함된다.

핵심 차이점: QUIC는 **연결 설정과 암호화 핸드셰이크를 통합**한다. TCP+TLS는 TCP 3-way handshake 후 TLS 핸드셰이크가 추가로 필요하여 다수의 RTT가 소요되지만, QUIC는 이를 결합하여 **더 빠른 연결 설정**을 제공한다.

### 2. 스트림 (Streams)

QUIC는 단일 연결 내에서 여러 애플리케이션 수준의 **"스트림"**을 멀티플렉싱한다. 연결이 설정되면 새 스트림을 빠르게 추가할 수 있다.

각 스트림은:
- 두 QUIC 엔드포인트 간의 **신뢰적, 순서 보장, 양방향** 데이터 전달 추상화
- **스트림 ID**로 식별
- 여러 스트림의 데이터가 하나의 QUIC 세그먼트에 담길 수 있음

HTTP/3에서는 웹 페이지의 각 객체마다 별도 스트림을 사용한다.

### 3. 신뢰적 전송 + HOL 블로킹 해결

QUIC는 **스트림별(per-stream)** 신뢰적 전송을 제공한다.

TCP의 문제: 단일 TCP 연결 위에 여러 HTTP 요청을 보내면(HTTP/1.1), TCP는 **바이트 순서 보장**을 하므로 하나의 세그먼트 손실이 모든 후속 데이터 전달을 차단한다 (**HOL(Head-of-Line) 블로킹**).

QUIC의 해결: 스트림별로 독립적인 순서 보장을 제공하므로, 한 스트림의 손실이 **다른 스트림에 영향을 주지 않는다**. 손실된 세그먼트와 같은 스트림의 데이터만 대기하면 된다.

QUIC의 신뢰적 전송은 TCP와 유사한 확인응답 메커니즘을 사용한다 (RFC 5681 참조).

### 4. 혼잡 제어

QUIC의 혼잡 제어는 **TCP NewReno**(RFC 6582) 기반이다. QUIC 초안 명세는 "TCP의 손실 감지 및 혼잡 제어에 익숙한 독자는 여기서 잘 알려진 TCP 알고리즘과 유사한 알고리즘을 발견할 것"이라고 명시한다.

## 애플리케이션 계층 프로토콜로서의 장점

QUIC는 전송 계층(커널)이 아닌 **애플리케이션 계층**에서 구현되므로, TCP나 UDP의 업데이트 주기보다 훨씬 빠른 **"애플리케이션 업데이트 타임스케일"**로 변경과 배포가 가능하다.

## 관련 프로토콜

- **[[TCP]]**: QUIC가 대체/개선하려는 전송 프로토콜. QUIC는 TCP의 신뢰적 전송과 혼잡 제어 원리를 계승
- **[[UDP]]**: QUIC의 하위 전송 프로토콜. QUIC는 UDP의 비신뢰적 데이터그램 서비스 위에 구축
- **[[HTTP]]**: HTTP/3는 QUIC 위에서 동작. HTTP/2의 스트림 멀티플렉싱과 QUIC의 스트림 개념이 결합

## 출처
- [[kurose-ch3]]
