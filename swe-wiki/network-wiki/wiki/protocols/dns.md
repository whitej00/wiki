---
tags: [protocol, application-layer]
layer: application
rfc: [1034, 1035]
related: [UDP, http, smtp]
source_count: 1
last_updated: 2026-04-08
---

# DNS (Domain Name System)

호스트명을 IP 주소로 변환하는 인터넷의 **디렉토리 서비스**. 분산 계층 데이터베이스와 애플리케이션 계층 프로토콜로 구성된다.

## 한 줄 요약

계층적으로 분산된 DNS 서버들이 호스트명-IP 주소 매핑을 저장하며, UDP 포트 53을 통해 질의/응답한다.

## DNS가 제공하는 서비스

1. **호스트명 → IP 주소 변환**: 핵심 기능
2. **호스트 엘리어싱(Host Aliasing)**: 복잡한 정규 호스트명에 대한 별칭 제공
3. **메일 서버 엘리어싱**: MX 레코드로 메일 서버 별칭 지원
4. **부하 분산(Load Distribution)**: 하나의 호스트명에 여러 IP를 매핑, 응답마다 순서 로테이션

## 분산 계층 데이터베이스

### 3단계 서버 계층

| 서버 유형 | 역할 | 특징 |
|---|---|---|
| **Root DNS** | TLD 서버의 IP 주소 제공 | 13개 루트 서버 (1000+ 인스턴스), 12개 조직이 관리 |
| **TLD DNS** | Authoritative 서버의 IP 주소 제공 | com, org, net, edu, 국가코드(.kr, .jp 등) |
| **Authoritative DNS** | 실제 호스트명-IP 매핑 보유 | 조직별로 자체 운영 또는 위탁 |

### Local DNS 서버
- ISP별로 운영 (기본 네임 서버라고도 함)
- DNS 계층에 속하지 않지만, 클라이언트의 DNS 질의를 대리 처리하는 **프록시** 역할
- DHCP로 호스트에 자동 할당

## 질의 방식

### 재귀 질의 (Recursive)
클라이언트 → Local DNS에게 "대신 알아봐 줘" 요청. Local DNS가 계층을 순회하며 최종 답을 가져옴.

### 반복 질의 (Iterative)
각 DNS 서버가 "나는 모르지만, 이 서버에 물어봐"라고 다음 서버 주소를 알려줌.

> **실제 패턴**: 클라이언트→Local DNS는 재귀, Local DNS→Root/TLD/Authoritative는 반복.

## DNS 캐싱

- DNS 서버가 응답을 받으면 매핑을 로컬 메모리에 **캐시**
- **TTL**(Time to Live) 후 만료 삭제 (보통 2일)
- 캐싱 덕분에 **루트 서버는 대부분의 질의에서 우회됨**
- Local DNS가 TLD 서버 IP도 캐시 → 더 빠른 응답

## 리소스 레코드 (Resource Record, RR)

형식: `(Name, Value, Type, TTL)`

| Type | Name | Value | 용도 |
|---|---|---|---|
| **A** | 호스트명 | IP 주소 | 호스트명→IP 매핑 |
| **NS** | 도메인 | Authoritative DNS 서버 호스트명 | 도메인 위임 |
| **CNAME** | 별칭 호스트명 | 정규 호스트명 | 호스트 엘리어싱 |
| **MX** | 별칭 호스트명 | 메일 서버 정규 호스트명 | 메일 서버 지정 |

## DNS 메시지 포맷

질의/응답 모두 동일한 포맷:
- **헤더** (12바이트): ID, 플래그(질의/응답, 재귀 요청/가능), 각 섹션 RR 수
- **Question 섹션**: 질의 이름 + 타입
- **Answer 섹션**: 질의에 대한 RR
- **Authority 섹션**: 권한 있는 서버의 RR
- **Additional 섹션**: 도움이 되는 추가 RR

## DNS 등록 과정

1. **등록 기관(Registrar)**에 도메인 등록 (ICANN 인가)
2. Authoritative DNS 서버의 NS 레코드와 A 레코드를 TLD 서버에 삽입
3. Authoritative DNS 서버에 호스트의 A 레코드, MX 레코드 등 설정

## 보안 취약점

- DDoS 공격 (루트 서버, TLD 서버 대상)
- 중간자 공격 (DNS 가로채기)
- **DNS 포이즈닝**: 위조 응답으로 캐시 오염
- **DNSSEC** (RFC 4033): DNS 보안 확장, 응답 인증/무결성 검증

## 출처

- [[kurose-ch2]] — Kurose 8판 2.4절
