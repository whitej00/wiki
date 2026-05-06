---
tags: [source, security]
source: "Kurose & Ross, Computer Networking: A Top-Down Approach, 8th Edition, Chapter 8"
pages: 637-712
related: [cryptography, message-integrity, end-point-authentication, tls, ipsec, pgp, firewall, ids-ips]
created: 2026-04-09
---

# Kurose 8판 8장: Security in Computer Networks

## 개요

네트워크 보안의 기본 원리(암호학, 메시지 무결성, 인증)와 이를 활용한 실제 보안 프로토콜(PGP, TLS, IPsec, 무선 보안), 운영 보안(방화벽, IDS/IPS)을 다룬다.

## 주요 토픽

### 8.1 네트워크 보안이란?
- 보안의 네 가지 요소: 기밀성(Confidentiality), 종단점 인증(End-point Authentication), 메시지 무결성(Message Integrity), 운영 보안(Operational Security)
- Alice(송신), Bob(수신), Trudy(침입자) 모델
- 침입자 능력: 도청(eavesdropping), 메시지 수정/삽입/삭제

### 8.2 암호화 원리
- **대칭키 암호**: Caesar cipher, 단일치환(monoalphabetic), 다중치환(polyalphabetic), 블록 암호(block cipher)
  - DES (56비트 키, 64비트 블록), 3DES, AES (128비트 블록, 128/192/256비트 키)
  - CBC (Cipher Block Chaining): IV + 이전 암호문 XOR로 동일 평문 블록 구별
- **공개키 암호**: RSA (소인수분해 난이도 기반), Diffie-Hellman (키 교환)
  - RSA: n=pq, 공개키 (n,e), 비밀키 (n,d), 암호화 c=m^e mod n, 복호화 m=c^d mod n
  - ed mod z = 1 (z=(p-1)(q-1))
- **세션 키**: RSA로 대칭키를 교환한 뒤 대칭키로 실제 데이터 암호화 (성능)
- 공격 유형: ciphertext-only, known-plaintext, chosen-plaintext

### 8.3 메시지 무결성과 디지털 서명
- **암호학적 해시 함수**: H(x)≠H(y) for x≠y를 찾기 어려움. MD5 (128비트), SHA-1 (160비트)
- **MAC (Message Authentication Code)**: 공유 비밀 s와 메시지 m을 결합 → H(m+s). HMAC 표준 (RFC 2104)
- **디지털 서명**: 개인키로 메시지 해시를 암호화 → K_B^-(H(m)). 검증 가능 + 위조 불가
- **공개키 인증서(PKC)**: CA(Certification Authority)가 공개키와 신원을 묶어 서명. X.509 표준
- MAC vs 디지털 서명: MAC은 대칭키 기반(가벼움), 디지털 서명은 PKI 필요(무거움, 부인방지 지원)

### 8.4 종단점 인증
- ap1.0 → ap4.0까지 프로토콜 진화
- ap1.0: 단순 선언 → Trudy 사칭 가능
- ap2.0: IP 주소 확인 → IP 스푸핑 가능
- ap3.0: 비밀번호 → 도청 가능
- ap3.1: 암호화된 비밀번호 → 재생 공격(playback attack)
- ap4.0: **nonce** (일회성 값) + 대칭키 암호화로 "라이브니스" 확인

### 8.5 이메일 보안
- 기밀성: 대칭키로 메시지 암호화, RSA로 세션키 전달
- 인증+무결성: 해시+개인키 서명
- 기밀성+인증+무결성 통합 설계
- **PGP**: 위 설계 구현. MD5/SHA 해시, CAST/3DES/IDEA 대칭키, RSA 공개키. Web of Trust 방식 PKC

### 8.6 TLS (Transport Layer Security)
- TCP 위에서 기밀성, 데이터 무결성, 서버 인증 제공
- 기술적으로 애플리케이션 계층에 위치하지만 전송 프로토콜처럼 동작
- **핸드셰이크**: TCP 연결 → TLS hello/인증서 교환 → PMS 생성 → MS 유도
- **키 유도**: MS에서 4개 키 생성 (E_B, M_B, E_A, M_A — 암호화 2 + HMAC 2)
- **데이터 전송**: 레코드 단위로 분할 → HMAC 추가(시퀀스 번호 포함) → 암호화
- **TLS 레코드**: Type | Version | Length | Data | HMAC (Data+HMAC 암호화)
- nonce로 연결 재생 공격 방어, 시퀀스 번호로 세그먼트 재생 방어
- **연결 종료**: type 필드로 closure 표시 → truncation attack 방지

### 8.7 IPsec과 VPN
- 네트워크 계층 보안: 기밀성, 출처 인증, 데이터 무결성, 재생 공격 방지 ("blanket coverage")
- **VPN**: 공용 인터넷 위에 IPsec으로 암호화된 사설 네트워크 구축
- AH(Authentication Header): 인증+무결성, 기밀성 없음. ESP(Encapsulation Security Payload): 인증+무결성+기밀성
- **SA (Security Association)**: 단방향 논리 연결. SPI, 암호화 알고리즘/키, MAC 알고리즘/키 포함. SAD에 저장
- **IPsec 데이터그램 (터널 모드)**: New IP Header | ESP Header(SPI, Seq#) | 원본 IP 헤더 | 원본 페이로드 | ESP Trailer | ESP MAC
  - "Enchilada": ESP Header~ESP Trailer는 인증 대상, 원본 IP 헤더~ESP Trailer는 암호화 대상
- **SPD (Security Policy Database)**: 어떤 트래픽을 IPsec 처리할지 결정
- **IKE (Internet Key Exchange)**: RFC 5996. Diffie-Hellman으로 IKE SA 설정 → IPsec SA 설정 (2단계)

### 8.8 무선 LAN 및 4G/5G 보안
- **802.11 보안**: 상호 인증 + 암호화가 핵심
  - WEP: 심각한 보안 결함 → WPA1 → WPA2 (AES) → WPA3 (2018)
  - 4단계: Discovery → 상호 인증/키 유도 → 세션키 배포 → 암호화 통신
  - WPA2 4-way handshake: 공유 비밀 K_AS-M → nonce 교환 → 세션키 K_M-AP 유도
  - 프로토콜 스택: EAP-TLS / EAP / EAPoL(무선) + RADIUS(유선)
- **4G LTE AKA**: IMSI + 공유 비밀 K_HSS-M → HSS가 auth_token/xres_HSS 생성 → 상호 인증 → 키 유도
- **5G 변경사항**: 홈 네트워크가 인증 결정, AKA' 프로토콜 추가, IMSI 암호화 전송 (공개키 사용)

### 8.9 운영 보안
- **방화벽**: 조직 네트워크와 인터넷 사이에서 트래픽 통제
  - Traditional packet filter: IP/포트/프로토콜/플래그 기반 필터링, ACL
  - Stateful packet filter: 연결 추적 테이블 유지, 외부 발 ACK 공격 차단
  - Application gateway: 애플리케이션 레벨 검사 (사용자 인증 등)
- **IDS/IPS**: 심층 패킷 검사(deep packet inspection)
  - Signature-based: 공격 시그니처 DB 매칭 (Snort). 신종 공격에 취약
  - Anomaly-based: 정상 트래픽 프로파일 학습 → 이상 탐지. 새로운 공격 탐지 가능
  - DMZ (비무장지대): 외부 접속 서버 배치

## 다룬 토픽 목록

| 토픽 | 위키 페이지 | 유형 |
|------|-----------|------|
| 암호화 원리 | [[cryptography]] | 개념 |
| 메시지 무결성 | [[message-integrity]] | 개념 |
| 종단점 인증 | [[end-point-authentication]] | 개념 |
| TLS | [[TLS]] | 프로토콜 |
| IPsec/VPN | [[IPsec]] | 프로토콜 |
| PGP | [[PGP]] | 프로토콜 |
| 방화벽 | [[firewall]] | 개념 |
| IDS/IPS | [[ids-ips]] | 개념 |
