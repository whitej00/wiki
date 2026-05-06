---
tags: [source, application]
source: "Kurose & Ross, Computer Networking: A Top-Down Approach, 8th Edition"
chapter: 2
title: "Application Layer"
pages: "111-210"
last_updated: 2026-04-08
---

# Kurose 8판 — Chapter 2: Application Layer

## 핵심 내용

애플리케이션 계층의 원리와 주요 프로토콜(HTTP, SMTP, DNS)을 다루고, P2P 파일 분배, 비디오 스트리밍과 CDN, 소켓 프로그래밍까지 포함하는 챕터.

## 다룬 토픽

### 2.1 네트워크 애플리케이션의 원리
- **애플리케이션 아키텍처**: 클라이언트-서버 vs P2P
- **프로세스 통신**: 소켓(Socket)은 프로세스와 네트워크 사이의 인터페이스 (API)
- **주소 지정**: IP 주소 + 포트 번호로 프로세스 식별
- **전송 서비스 4차원**: 신뢰성, 처리량, 타이밍, 보안
- **TCP 서비스**: 연결 지향, 신뢰적 데이터 전송, 혼잡 제어
- **UDP 서비스**: 비연결, 비신뢰적, 최소한의 서비스
- **TLS**: TCP 위에서 암호화/무결성/인증 제공 (애플리케이션 계층에서 구현)

### 2.2 [[HTTP]]
- HTTP/1.0: 비지속 연결 (객체당 2 RTT)
- HTTP/1.1: 지속 연결 + 파이프라이닝 (기본)
- **HTTP 메시지 포맷**: 요청(method, URL, headers, body) / 응답(status code, headers, body)
- **쿠키(Cookie)**: 무상태 HTTP에 사용자 상태 유지 (Set-cookie / Cookie 헤더)
- **웹 캐시(프록시 서버)**: 응답 시간 감소, 트래픽 감소, Conditional GET (If-Modified-Since → 304 Not Modified)
- **HTTP/2**: 단일 [[TCP]] 연결에서 HOL 블로킹 해결 (프레이밍, 인터리빙, 우선순위, 서버 푸시)
- **HTTP/3**: [[QUIC]] ([[UDP]] 기반) 위에서 동작

### 2.3 전자 메일
- 3대 구성 요소: 사용자 에이전트, 메일 서버, **[[SMTP]]**
- **SMTP** (RFC 5321): [[TCP]] 포트 25, push 프로토콜, 7비트 ASCII 제한, 지속 연결
- **메일 접근 프로토콜**: IMAP (RFC 3501), HTTP (Gmail 등)

### 2.4 [[DNS]]
- **DNS 서비스**: 호스트명→IP 변환, 호스트 엘리어싱, 메일 서버 엘리어싱, 부하 분산
- **분산 계층 데이터베이스**: Root → TLD → Authoritative DNS 서버
- **Local DNS 서버**: ISP별 존재, 재귀+반복 질의 조합
- **DNS 캐싱**: TTL 기반, 루트 서버 우회 가능
- **리소스 레코드(RR)**: (Name, Value, Type, TTL) — A, NS, CNAME, MX 타입
- **DNS 메시지**: 질의/응답 동일 포맷, 12바이트 헤더 + Question/Answer/Authority/Additional 섹션

### 2.5 P2P 파일 분배
- **자기 확장성(Self-scalability)**: 피어가 늘어날수록 총 업로드 용량도 증가
- 분배 시간: CS = max{NF/u_s, F/d_min}, P2P = max{F/u_s, F/d_min, NF/(u_s + Σu_i)}
- **BitTorrent**: 토렌트, 트래커, 256KB 청크, rarest first, tit-for-tat (unchoke/optimistic unchoke)

### 2.6 비디오 스트리밍과 [[cdn|CDN]]
- **DASH** (Dynamic Adaptive Streaming over HTTP): 다중 비트레이트 인코딩, manifest 파일, 적응적 품질 선택
- **CDN**: Enter Deep (ISP 내부에 소규모 클러스터) vs Bring Home (IXP에 대규모 클러스터)
- CDN 동작: [[DNS]] 리다이렉트로 클러스터 선택
- 사례: Netflix (자체 CDN, Amazon 클라우드, push 캐싱), YouTube (Google CDN, pull 캐싱)

### 2.7 소켓 프로그래밍
- **UDP 소켓**: SOCK_DGRAM, sendto/recvfrom, 목적지 주소 명시 필요
- **TCP 소켓**: SOCK_STREAM, connect/accept, welcoming socket + connection socket, 바이트 스트림

## 위키에 반영된 페이지

- [[http]] — HTTP 프로토콜
- [[smtp]] — SMTP 프로토콜
- [[dns]] — DNS 프로토콜
- [[cdn]] — 콘텐츠 분배 네트워크
- [[layers/application]] — 애플리케이션 계층 개관

## 출처

Kurose, J. F. & Ross, K. W. (2022). *Computer Networking: A Top-Down Approach* (8th Global Edition). Pearson. Chapter 2.
