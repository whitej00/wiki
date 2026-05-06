---
tags: [concept, security]
layer: application
related: [cryptography, message-integrity, tls, ipsec]
source_count: 1
last_updated: 2026-04-09
---

# 종단점 인증 (End-Point Authentication)

네트워크를 통해 통신하는 한 엔티티가 상대방의 신원을 **실시간으로** 확인하는 과정. 과거 시점에 메시지가 특정 송신자로부터 왔음을 증명하는 디지털 서명과는 다르게, 상대방이 **지금 실제로 존재하는지(liveness)** 를 확인한다.

## 왜 어려운가?

네트워크에서는 생체 정보(얼굴, 음성)를 사용할 수 없다. 오직 **메시지와 데이터 교환**만으로 인증해야 한다. 라우터, 클라이언트/서버 등 네트워크 요소 간 인증에 필수적이다.

## 인증 프로토콜의 진화

교재에서는 결함 있는 프로토콜을 하나씩 개선해 나가는 방식으로 설명한다:

### ap1.0 — 단순 선언

```
Alice → Bob: "I am Alice"
```

**결함**: Trudy도 "I am Alice"를 보낼 수 있다. 검증 수단 없음.

### ap2.0 — IP 주소 확인

```
Alice → Bob: "I am Alice" + Alice의 IP 주소
```

**결함**: **IP 스푸핑(IP Spoofing)**으로 Trudy가 Alice의 IP 주소를 위조할 수 있다 (RFC 2827). 리눅스 등에서 원하는 소스 IP로 데이터그램 생성 가능.

### ap3.0 — 비밀번호

```
Alice → Bob: "I am Alice" + 비밀번호
Bob → Alice: OK
```

**결함**: Trudy가 **도청(eavesdropping)**하여 비밀번호를 탈취. Telnet 등에서 비밀번호가 평문 전송되는 실제 문제.

### ap3.1 — 암호화된 비밀번호

```
Alice → Bob: "I am Alice" + K_{A-B}(비밀번호)
Bob → Alice: OK
```

**결함**: **재생 공격(Playback Attack)**. Trudy가 암호화된 비밀번호를 녹음하여 그대로 재전송. Bob은 원래 인증과 재생을 구별할 수 없다.

### ap4.0 — Nonce + 대칭키 (최종)

```
1. Alice → Bob: "I am Alice"
2. Bob → Alice: R (nonce)
3. Alice → Bob: K_{A-B}(R)
4. Bob: 복호화하여 R 확인 → 인증 완료
```

## Nonce

**nonce**: 프로토콜이 **일생에 단 한 번만** 사용하는 값. "number used once"의 준말.

nonce가 해결하는 문제:
- **라이브니스(Liveness)**: Bob이 방금 생성한 R을 Alice가 암호화해서 돌려보냈으므로, Alice가 **지금** 통신하고 있음을 확인
- **재생 공격 방지**: R은 매번 다르므로 이전 세션의 응답을 재사용할 수 없음

> TCP 3-way handshake의 초기 순서 번호도 동일한 원리 — 상대방의 "라이브니스"를 확인하기 위해 랜덤 값을 사용한다.

## 공개키 기반 인증

nonce + 공개키 암호를 사용하는 변형도 가능하지만, 교재에서는 연습 문제로 남겨둔다. TLS에서는 이 방식(인증서 + nonce)을 실제로 사용한다.

## 출처

- [kurose-ch8](../sources/kurose-ch8.md) — Kurose 8판 8.4절
