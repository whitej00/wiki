---
tags: [protocol, security, transport-layer]
layer: transport
rfc: 4346
related: [TCP, cryptography, message-integrity, end-point-authentication, HTTP, ipsec]
source_count: 1
last_updated: 2026-04-09
---

# TLS (Transport Layer Security)

TCP에 기밀성(Confidentiality), 데이터 무결성(Data Integrity), 서버 인증(Server Authentication)을 추가한 보안 프로토콜. SSL 3.0의 후속으로 IETF에서 표준화 (RFC 4346).

## 한 줄 요약

TCP 위에서 동작하며 암호화된 통신 채널을 제공하는 사실상의 인터넷 보안 표준 (HTTPS = HTTP over TLS).

## 프로토콜 스택에서의 위치

기술적으로 **애플리케이션 계층**에 위치하지만, 개발자 관점에서는 보안이 강화된 **전송 프로토콜**처럼 동작한다.

```
┌─────────────┐
│ Application  │
├─────────────┤
│ TLS socket   │ ← TLS sublayer
├─────────────┤
│ TCP socket   │
├─────────────┤
│ TCP          │
├─────────────┤
│ IP           │
└─────────────┘
```

TLS는 TCP의 소켓 API와 유사한 **SSL 클래스/라이브러리**를 통해 애플리케이션에 제공된다.

## TLS가 없다면?

전자상거래 시나리오에서:
- 기밀성 없음 → 신용카드 정보 탈취
- 무결성 없음 → 주문 수량 변조
- 서버 인증 없음 → 가짜 사이트에 정보 제공 (피싱)

## 세 단계

### 1. 핸드셰이크 (Handshake)

TCP 연결 위에서 보안 파라미터를 협상하고, 공유 비밀을 설정한다.

**Almost-TLS (간소화 버전)**:
1. Bob → Alice: TLS hello
2. Alice → Bob: 인증서 (공개키 포함)
3. Bob: CA 인증서로 Alice의 공개키 검증 → Master Secret(MS) 생성 → EMS = K_A^+(MS) 전송
4. Alice: 개인키로 EMS 복호화 → MS 획득

**실제 TLS 핸드셰이크**:
1. 클라이언트 → 서버: 지원 암호 알고리즘 목록 + **client nonce**
2. 서버 → 클라이언트: 선택된 알고리즘(대칭키: AES, 공개키: RSA, HMAC: MD5/SHA-1) + 인증서 + **server nonce**
3. 클라이언트: 인증서 검증 → **Pre-Master Secret (PMS)** 생성 → 서버 공개키로 암호화하여 전송
4. 클라이언트 & 서버: PMS + nonces로 독립적으로 **Master Secret (MS)** 계산
5. 클라이언트: 전체 핸드셰이크 메시지의 HMAC 전송
6. 서버: 전체 핸드셰이크 메시지의 HMAC 전송

> 5, 6단계가 핸드셰이크 변조(Trudy가 약한 알고리즘만 남기는 공격)를 방지한다.

### 2. 키 유도 (Key Derivation)

MS에서 **4개의 키**를 유도:

| 키 | 용도 |
|---|------|
| E_B | Bob→Alice 데이터 **암호화** 키 |
| M_B | Bob→Alice **HMAC** 키 |
| E_A | Alice→Bob 데이터 **암호화** 키 |
| M_A | Alice→Bob **HMAC** 키 |

CBC 사용 시 양방향 각각의 **초기화 벡터(IV)**도 MS에서 유도한다.

> 암호화와 무결성에 **별도의 키**를, 양방향에 **별도의 키**를 사용하는 것이 보안 모범 관행이다.

### 3. 데이터 전송 (Data Transfer)

TCP는 바이트 스트림이므로, TLS는 이를 **레코드(Record)** 단위로 분할하여 처리한다:

1. 데이터를 레코드로 분할
2. 각 레코드에 HMAC 추가 (HMAC 키 M + **시퀀스 번호** 포함)
3. 레코드 + HMAC을 세션 암호화 키 E로 암호화
4. 암호화된 결과를 TCP에 전달

> **시퀀스 번호**는 레코드에 직접 포함되지 않지만, HMAC 계산에 포함되어 재정렬/재생 공격을 방지한다.

## TLS 레코드 형식

```
┌──────┬─────────┬────────┬──────────────────┬──────┐
│ Type │ Version │ Length │      Data        │ HMAC │
└──────┴─────────┴────────┴──────────────────┴──────┘
                          ├── 암호화 (키 E) ──────────┤
```

- **Type**: 핸드셰이크 / 애플리케이션 데이터 / **closure** (연결 종료)
- Type, Version, Length는 암호화되지 않음 (수신측이 레코드 경계를 파악해야 하므로)

## Nonce의 역할

| 공격 | 방어 수단 |
|------|----------|
| **세그먼트 재생** (세션 내) | 시퀀스 번호 (HMAC에 포함) |
| **연결 재생** (전체 세션 재생) | 핸드셰이크의 **nonce** → 매 세션마다 다른 키 유도 |

## 연결 종료 (Connection Closure)

단순히 TCP FIN을 보내면 **truncation attack** 위험:
- Trudy가 중간에 TCP FIN을 보내 세션을 조기 종료 → Alice는 Bob의 데이터를 일부만 수신

**해결**: TLS 레코드의 **type 필드**에 closure 표시. type 필드는 HMAC으로 인증되므로, TLS closure 레코드 없이 TCP FIN이 오면 비정상 종료로 판단.

## 출처

- [[kurose-ch8]] — Kurose 8판 8.6절
