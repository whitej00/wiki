---
tags: [protocol, security, application-layer]
layer: application
related: [cryptography, message-integrity, SMTP]
source_count: 1
last_updated: 2026-04-09
---

# PGP (Pretty Good Privacy)

1991년 Phil Zimmermann이 개발한 이메일 암호화 프로토콜. 기밀성, 송신자 인증, 메시지 무결성을 제공하며, 클라이언트/서버 애플리케이션 코드만으로 구현 가능한 최초의 널리 사용된 인터넷 보안 기술 중 하나이다.

## 한 줄 요약

대칭키 암호 + 공개키 암호 + 해시 + 디지털 서명을 결합하여 이메일의 기밀성·인증·무결성을 동시에 제공하는 애플리케이션 계층 보안 프로토콜.

## 사용 알고리즘

| 기능 | 알고리즘 |
|------|----------|
| 메시지 다이제스트 | MD5 또는 SHA |
| 대칭키 암호화 | CAST, triple-DES, 또는 IDEA |
| 공개키 암호화 | RSA |

## 동작 원리

### 기밀성만 필요한 경우

1. Alice가 랜덤 대칭 **세션 키** K_S 생성
2. K_S로 메시지 m 암호화: K_S(m)
3. Bob의 공개키로 세션 키 암호화: K_B^+(K_S)
4. K_S(m) + K_B^+(K_S)를 패키지로 전송
5. Bob: 개인키 K_B^-로 K_S 복호화 → K_S로 메시지 복호화

### 인증 + 무결성만 필요한 경우

1. Alice가 메시지 m에 해시 적용: H(m)
2. 자신의 개인키로 해시 서명: K_A^-(H(m)) → 디지털 서명
3. m + K_A^-(H(m))을 전송
4. Bob: Alice의 공개키 K_A^+로 서명 검증, 직접 계산한 H(m)과 비교

### 기밀성 + 인증 + 무결성 (완전 보안)

위 두 과정을 결합:
1. m + K_A^-(H(m))으로 인증·무결성 패키지 생성
2. 이 패키지를 세션 키 K_S로 암호화
3. K_S를 K_B^+로 암호화
4. 암호화된 패키지 + 암호화된 K_S를 전송

> Alice는 공개키 암호를 **두 번** 사용: 자신의 개인키(서명)와 Bob의 공개키(세션키 암호화). Bob도 두 번 사용: 자신의 개인키(세션키 복호화)와 Alice의 공개키(서명 검증).

## PGP 메시지 형식

### 서명된 메시지

```
-----BEGIN PGP SIGNED MESSAGE-----
Hash:  SHA1
Bob:
Can I see you tonight?
Passionately yours, Alice
-----BEGIN PGP SIGNATURE-----
Version: PGP for Personal Privacy 5.0
Charset: noconv
yhHJRHhGJGhgg/12EpJ+lo8gE4vB3mqJhFEvZP9t6n7G6m5Gw2
-----END PGP SIGNATURE-----
```

### 암호화된 메시지

```
-----BEGIN PGP MESSAGE-----
Version: PGP for Personal Privacy 5.0
u2R4d+/jKmn8Bc5+hgDsqAewsDfrGdszX68liKm5F6Gc4sDfcXyt
RfdS10juHgbcfDssWe7/K=lKhnMikLo0+1/BvcX4t==Ujk9PbcD4
Thdf2awQfgHbnmKlok8iy6gThlp
-----END PGP MESSAGE
```

## 공개키 인증: Web of Trust

PGP는 중앙화된 CA 대신 **신뢰의 그물(Web of Trust)** 모델을 사용:

- Alice가 직접 Bob의 공개키와 신원을 확인하고 서명(인증)할 수 있음
- Alice가 신뢰하는 Carol이 Bob을 인증했다면, Alice도 Bob을 신뢰 가능
- **키 서명 파티(Key-signing Party)**: 사용자들이 물리적으로 모여 서로의 공개키를 교환하고 개인키로 서명

> 기존 CA(X.509) 방식과 근본적으로 다른 분산형 인증 모델이다.

## 출처

- [[kurose-ch8]] — Kurose 8판 8.5절
