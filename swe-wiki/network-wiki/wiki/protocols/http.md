---
tags: [protocol, application-layer]
layer: application
rfc: [1945, 7230, 7540]
related: [TCP, DNS, TLS, cdn]
source_count: 1
last_updated: 2026-04-08
---

# HTTP (HyperText Transfer Protocol)

웹(World Wide Web)의 애플리케이션 계층 프로토콜. 클라이언트(브라우저)와 서버 간 웹 객체 요청/응답을 정의한다.

## 한 줄 요약

TCP 위에서 동작하는 **무상태(stateless) 요청-응답** 프로토콜로, 웹 페이지와 객체를 전송한다.

## 핵심 특성

- **클라이언트-서버 모델**: 브라우저(클라이언트)가 요청, 웹 서버가 응답
- **TCP 사용**: 기본 포트 80 (HTTPS는 443)
- **무상태(Stateless)**: 서버가 클라이언트 상태를 기억하지 않음
- **쿠키(Cookie)**: 무상태 위에 세션 계층을 구현 (Set-cookie / Cookie 헤더, RFC 6265)

## 연결 방식

### 비지속 연결 (Non-persistent, HTTP/1.0)
- 객체마다 별도의 TCP 연결
- 객체당 **2 RTT** 소요 (TCP 핸드셰이크 1 RTT + 요청/응답 1 RTT)
- 11개 객체 → 11개 TCP 연결

### 지속 연결 (Persistent, HTTP/1.1 기본)
- 하나의 TCP 연결로 여러 요청/응답 전송
- 파이프라이닝: 응답을 기다리지 않고 연속 요청 가능
- 서버가 일정 시간 미사용 시 연결 종료

## HTTP 메시지 포맷

### 요청 메시지
```
GET /somedir/page.html HTTP/1.1
Host: www.example.com
Connection: close
User-agent: Mozilla/5.0
Accept-language: ko
```
- **Request line**: 메서드(GET, POST, HEAD, PUT, DELETE) + URL + 버전
- **Header lines**: Host, Connection, User-agent 등
- **Entity body**: POST 메서드 시 폼 데이터 포함

### 응답 메시지
```
HTTP/1.1 200 OK
Date: Tue, 18 Aug 2015 15:44:04 GMT
Server: Apache/2.2.3
Content-Length: 6821
Content-Type: text/html
```
- **Status line**: 버전 + 상태 코드 + 문구
- **주요 상태 코드**: 200 OK, 301 Moved Permanently, 400 Bad Request, 404 Not Found, 505 HTTP Version Not Supported

## 웹 캐싱 (프록시 서버)

- 오리진 서버를 대신하여 HTTP 요청을 처리하는 네트워크 엔티티
- 클라이언트 응답 시간 감소 + 기관 액세스 링크 트래픽 감소
- **Conditional GET**: `If-Modified-Since` 헤더로 캐시 유효성 검증 → 변경 없으면 `304 Not Modified` (본문 없이 응답)
- **CDN**(Content Distribution Network)이 웹 캐시를 대규모로 활용 → [[cdn]] 참조

## HTTP/2 (RFC 7540)

HTTP/1.1의 **HOL(Head-of-Line) 블로킹** 문제를 해결:
- 단일 TCP 연결에서 메시지를 **프레임** 단위로 분할하여 인터리빙
- **바이너리 프레이밍**: 더 효율적 파싱, 더 작은 프레임
- **응답 우선순위**: 1~256 가중치 기반 스케줄링
- **서버 푸시**: 클라이언트 요청 전에 필요한 객체를 선제 전송

## HTTP/3

- **QUIC** (UDP 기반) 위에서 동작
- HTTP/2의 메시지 인터리빙, 흐름 제어, 저지연 연결 설정 등을 애플리케이션 계층에서 구현
- 2020년 기준 인터넷 드래프트 단계

## 출처

- [[kurose-ch2]] — Kurose 8판 2.2절, 2.2.6절
