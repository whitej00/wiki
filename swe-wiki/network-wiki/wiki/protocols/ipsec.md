---
tags: [protocol, security, network-layer]
layer: network
rfc: [4301, 6071]
related: [IPv4, IPv6, tls, cryptography, message-integrity, nat]
source_count: 1
last_updated: 2026-04-09
---

# IPsec (IP Security)

네트워크 계층에서 IP 데이터그램을 보호하는 프로토콜 스위트. 호스트 간, 라우터 간, 또는 호스트-라우터 간 **기밀성, 출처 인증, 데이터 무결성, 재생 공격 방지**를 제공한다.

## 한 줄 요약

네트워크 계층에서 "blanket coverage" 보안을 제공하며, VPN 구축의 핵심 기술이다.

## VPN (Virtual Private Network)

조직의 지사 간 트래픽을 공용 인터넷 위에서 IPsec으로 암호화하여 사설 네트워크처럼 동작시킨다.

```
[본사]                                    [지사]
 호스트 ─── R1(게이트웨이) ═══ 인터넷 ═══ R2(게이트웨이) ─── 호스트
           IPsec 암호화 →           ← IPsec 복호화
```

- 같은 사이트 내부 통신: 일반 IPv4
- 사이트 간 통신: IPsec 암호화
- 인터넷 일반 접속(Amazon, Google): 일반 IPv4

## 프로토콜: AH vs ESP

| | AH (Authentication Header) | ESP (Encapsulation Security Payload) |
|---|---|---|
| 출처 인증 | O | O |
| 데이터 무결성 | O | O |
| 기밀성 | **X** | **O** |
| 사용 빈도 | 낮음 | **높음** (VPN의 표준) |

교재에서는 ESP에 집중한다.

## SA (Security Association)

IPsec 데이터그램을 보내기 전에 송신-수신 간 설정하는 **단방향(Simplex) 논리 연결**.

양방향 통신에는 **2개의 SA**가 필요 (각 방향 하나씩). n명의 원격 사용자가 있으면 총 (2 + 2n)개 SA.

### SA 상태 정보

- **SPI (Security Parameter Index)**: 32비트 SA 식별자
- 출발지/목적지 인터페이스 IP
- 암호화 알고리즘 + 키 (예: 3DES with CBC)
- 무결성 검사 알고리즘 + 키 (예: HMAC with MD5)

### SAD (Security Association Database)

OS 커널에 유지되는 모든 SA 정보의 저장소. 수신 시 SPI로 SAD를 조회하여 복호화/인증 파라미터를 결정한다.

### SPD (Security Policy Database)

"무엇을 할지" 결정: 소스 IP, 목적지 IP, 프로토콜 타입에 따라 해당 데이터그램을 IPsec 처리할지, 어떤 SA를 사용할지 판단한다.

- SPD = "what" to do
- SAD = "how" to do it

## IPsec 데이터그램 (터널 모드)

```
"Enchilada" authenticated
├────────────────────────────────────────────────────────┤
│              Encrypted                                  │
│  ┌────────────────────────────────────────────┐         │
┌────────┬────────────┬──────────┬───────────────┬──────────┬─────────┐
│ New IP │ ESP Header │ Original │ Original IP   │ ESP      │ ESP     │
│ Header │ (SPI,Seq#) │ IP Hdr   │ Datagram      │ Trailer  │ MAC     │
│        │            │          │ Payload       │          │         │
└────────┴────────────┴──────────┴───────────────┴──────────┴─────────┘
```

### 구성 과정 (R1 → R2 예시)

1. 원본 IPv4 데이터그램 뒤에 **ESP 트레일러** 추가 (패딩, 패드 길이, next header)
2. SA의 알고리즘/키로 [원본 IP 헤더 + 페이로드 + ESP 트레일러]를 **암호화**
3. 암호화된 데이터 앞에 **ESP 헤더** (SPI + 시퀀스 번호) 추가 → "enchilada"
4. [ESP 헤더 + 암호화 데이터] 전체에 대해 **MAC** 계산 → 뒤에 추가
5. **새 IP 헤더** 생성: 소스=R1, 목적지=R2, 프로토콜=50(ESP)

### 수신 처리 (R2)

1. 프로토콜 번호 50 확인 → ESP 처리
2. SPI로 SAD 조회 → SA 확인
3. MAC 검증 → 무결성 확인
4. 시퀀스 번호 확인 → 재생 공격 방지
5. 복호화 → 패딩 제거 → 원본 IPv4 데이터그램 추출
6. 원본 데이터그램을 내부 네트워크로 포워딩

### 보안 서비스 요약

Trudy가 R1-R2 사이에서 할 수 있는 것과 없는 것:
- **원본 데이터 확인**: 불가 (암호화)
- **프로토콜/소스IP/목적지IP 확인**: 불가 (암호화됨, 터널 모드)
- **변조**: 불가 (MAC 검증 실패)
- **위장 (R1 사칭)**: 불가 (MAC 검증 실패)
- **재생**: 불가 (시퀀스 번호)

> TLS보다 더 강력한 기밀성 — IPsec은 원본 IP 주소까지 숨긴다.

## IKE (Internet Key Exchange)

대규모 VPN에서 SA를 수동으로 설정하는 것은 비현실적. **IKE** (RFC 5996)가 자동으로 SA를 설정한다.

### 2단계 프로세스

**Phase 1**: Diffie-Hellman으로 **IKE SA** 설정 (양방향). 인증된 암호화 채널 확립. 이 단계에서는 공개키/개인키로 신원 공개하지 않음.

**Phase 2**: IKE SA 위에서 메시지 교환하여 **IPsec SA** 설정 (각 방향). 신원 공개(서명). 암호화/인증 알고리즘 협상. 공개키 암호 불필요 → 다수 SA를 저비용으로 생성 가능.

## 출처

- [kurose-ch8](../sources/kurose-ch8.md) — Kurose 8판 8.7절
