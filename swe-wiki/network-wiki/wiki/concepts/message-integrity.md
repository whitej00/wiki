---
tags: [concept, security]
layer: application
related: [cryptography, end-point-authentication, tls, ipsec, pgp]
source_count: 1
last_updated: 2026-04-09
---

# 메시지 무결성 (Message Integrity)

메시지가 전송 중 변조되지 않았음을 검증하고, 메시지의 출처를 확인하는 기술. 기밀성(Encryption)과는 독립적인 보안 요구사항이다.

## 왜 필요한가?

- 라우팅 프로토콜(예: [[OSPF]])에서 위조된 link-state 메시지가 주입되면 라우팅 테이블 전체가 오염
- 이메일, 전자상거래 등에서 메시지가 변조되었는지 검증 필요
- 기밀성이 불필요한 경우에도 무결성은 여전히 필요

## 암호학적 해시 함수 (Cryptographic Hash Function)

임의 길이 메시지 m을 **고정 길이 해시값** H(m)으로 변환하는 함수.

**핵심 성질**: H(x) = H(y)인 서로 다른 x, y를 찾는 것이 **계산적으로 불가능**(collision resistance)

> 인터넷 체크섬(Internet checksum)은 암호학적 해시가 아니다. "IOU100.99BOB"과 "IOU900.19BOB"이 동일한 체크섬을 가지는 예시가 있다.

### 주요 알고리즘

| 알고리즘 | 출력 길이 | 비고 |
|----------|----------|------|
| **MD5** | 128비트 | RFC 1321. 패딩 → 길이 추가 → 초기화 → 4라운드 처리 |
| **SHA-1** | 160비트 | FIPS 1995. US 연방 표준. MD5보다 안전 |

## MAC (Message Authentication Code)

해시 함수 + **공유 비밀(authentication key)** s를 결합하여 메시지 무결성을 제공한다.

### 동작 과정

1. Alice: m에 s를 결합(concatenate)하여 MAC = H(m + s) 계산
2. Alice → Bob: (m, H(m + s)) 전송
3. Bob: s를 알고 있으므로 H(m + s)를 재계산하여 비교

> 단순히 H(m)만 보내는 것은 무의미 — Trudy가 m'을 만들어 (m', H(m'))을 보낼 수 있음

### HMAC

MAC의 가장 널리 사용되는 표준. MD5 또는 SHA-1과 함께 사용하며, 해시 함수를 **두 번** 통과시킨다 (RFC 2104).

### MAC의 특징

- 암호화 알고리즘 **불필요** → 가벼움
- OSPF, TLS, IPsec 등에서 광범위하게 사용
- 인증 키 배포 문제: 네트워크 관리자가 물리적으로 배포하거나, 공개키 암호로 암호화하여 전송

> **주의**: 여기서 MAC는 "Message Authentication Code"이며, 링크 계층의 "Medium Access Control"과 전혀 다르다!

## 디지털 서명 (Digital Signature)

공개키 암호를 사용하여 **검증 가능(verifiable)** + **위조 불가(nonforgeable)** + **부인 방지(nonrepudiation)** 를 제공한다.

### 서명 과정

1. Bob이 메시지 m의 해시를 계산: H(m)
2. **개인키**로 해시를 암호화: K_B^-(H(m)) → 이것이 디지털 서명
3. 원본 메시지 m + 서명 K_B^-(H(m))을 함께 전송

> 메시지 전체가 아닌 **해시를 서명**하는 이유: 해시가 원본보다 훨씬 작아서 연산 비용 대폭 절감

### 검증 과정

1. Alice가 Bob의 **공개키** K_B^+로 서명을 복호화 → H(m) 획득
2. 수신한 메시지 m에 같은 해시 함수를 적용 → H(m) 계산
3. 두 값이 일치하면 → 무결성 + 출처 확인 완료

### MAC vs 디지털 서명

| | MAC | 디지털 서명 |
|---|-----|-----------|
| 키 | 대칭 (공유 비밀) | 비대칭 (개인키/공개키) |
| PKI 필요 | 아니오 | 예 (CA 인증서) |
| 부인 방지 | 불가 | 가능 |
| 연산 비용 | 낮음 | 높음 |
| 사용 예 | OSPF, TLS, IPsec | PGP, 전자상거래, 인증서 |

## 공개키 인증 (Public Key Certification)

공개키가 실제로 해당 엔티티의 것인지 검증하는 메커니즘. 없으면 Trudy가 자신의 공개키를 Bob의 것이라 속일 수 있다 ("피자 프랭크" 시나리오).

### CA (Certification Authority)

1. 엔티티의 신원을 검증
2. **인증서(Certificate)** 생성: 공개키 + 신원 정보를 CA의 개인키로 서명
3. 인증서를 제시받은 측은 CA의 공개키로 검증

### X.509 인증서 주요 필드

| 필드 | 설명 |
|------|------|
| Version | X.509 사양 버전 |
| Serial number | CA가 부여한 고유 식별자 |
| Signature | CA가 사용한 서명 알고리즘 |
| Issuer name | CA 식별 (DN 형식, RFC 4514) |
| Validity period | 인증서 유효 기간 |
| Subject name | 공개키 소유자 식별 (DN 형식) |
| Subject public key | 공개키 + 알고리즘 정보 |

## 출처

- [[kurose-ch8]] — Kurose 8판 8.3절
