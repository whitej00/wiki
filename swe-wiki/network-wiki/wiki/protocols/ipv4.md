---
tags: [protocol, network-layer]
layer: network
rfc: 791
related: [IPv6, DHCP, NAT, ip-addressing, TCP, UDP, ICMP]
source_count: 1
last_updated: 2026-04-08
---

# IPv4 (Internet Protocol version 4)

RFC 791에 정의된 인터넷의 핵심 네트워크 계층 프로토콜. **32비트 주소**를 사용하여 호스트 간 데이터그램을 전달하는 **비연결형, 비신뢰적** 프로토콜이다.

## IPv4 데이터그램 포맷

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |Type of Service|         Total Length          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|    Fragment Offset      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |        Header Checksum        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Source IP Address                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                 Destination IP Address                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (if any)                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         Data                                 |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### 주요 필드

- **Version** (4비트): IP 버전. IPv4 = 4
- **Header Length (IHL)** (4비트): 헤더 길이 (32비트 워드 단위). 옵션 없으면 5 → 20바이트
- **Type of Service (TOS)** (8비트): 서비스 유형 구분. 2비트는 **ECN**([혼잡 제어](../concepts/congestion-control.md))에 사용
- **Total Length** (16비트): 데이터그램 전체 길이 (바이트). 최대 65,535바이트. 통상 ≤ 1,500바이트 (이더넷 MTU)
- **Identification, Flags, Fragment Offset**: IP 단편화(fragmentation) 관련. 큰 데이터그램을 MTU에 맞게 분할
- **TTL (Time to Live)** (8비트): 라우터 통과 시마다 1 감소. 0이 되면 폐기 → 라우팅 루프 방지
- **Protocol** (8비트): 상위 계층 프로토콜 식별. TCP=6, UDP=17. 포트 번호가 전송↔애플리케이션을 연결하듯, 프로토콜 번호가 네트워크↔전송을 연결
- **Header Checksum** (16비트): 헤더 오류 검출. 1의 보수 합. **매 홉마다 재계산** (TTL 변경 때문)
- **Source/Destination IP Address** (각 32비트): 출발지/목적지 IP 주소
- **Options**: 가변 길이. 타임스탬프, 경로 기록 등. 실무에서 거의 사용되지 않으며, IPv6에서 제거됨
- **Data (Payload)**: 통상 전송 계층 세그먼트 (TCP 또는 UDP). ICMP 메시지도 가능

헤더 크기: 통상 **20바이트** (옵션 없을 때). TCP 세그먼트를 담으면 총 40바이트 오버헤드 (IP 20 + TCP 20).

### IP 단편화 (Fragmentation)

링크마다 **MTU(Maximum Transmission Unit)**가 다르므로, 데이터그램이 MTU보다 크면 **단편(fragment)**으로 분할한다. 재조립은 **목적지 호스트**에서만 수행한다 (중간 라우터는 하지 않음).

Identification, Flags(DF/MF), Fragment Offset 필드로 단편을 식별하고 재조립한다.

IPv6에서는 **라우터의 단편화가 금지**되어, "Packet Too Big" ICMP 메시지로 송신자에게 알린다.

### 헤더 체크섬

TCP/UDP 체크섬과 별도로 IP 헤더에 대한 체크섬이 존재한다:
- IP 체크섬은 **헤더만** 검사 (데이터 미포함)
- TCP/UDP 체크섬은 **전체 세그먼트** 검사
- TTL이 매 홉마다 변하므로 **모든 라우터에서 재계산** 필요 → 비용이 큼
- IPv6에서는 이 체크섬이 **제거**됨

## IP 주소

→ 상세: [ip-addressing](../concepts/ip-addressing.md)

- 32비트, **인터페이스** 단위 할당
- [서브넷](../concepts/ip-addressing.md), [CIDR](../concepts/ip-addressing.md), [최장 프리픽스 매칭](../concepts/ip-addressing.md) 참조
- 주소 획득: [DHCP](dhcp.md)
- 사설 주소와 공인 주소 변환: [NAT](../concepts/nat.md)

## 관련 프로토콜

- **[IPv6](ipv6.md)**: IPv4의 후속. 주소 고갈 문제 해결 (128비트)
- **[DHCP](dhcp.md)**: IPv4 주소 자동 할당
- **[NAT](../concepts/nat.md)**: 사설 IP ↔ 공인 IP 변환
- **[TCP](tcp.md), [UDP](udp.md)**: IPv4 데이터그램의 페이로드로 전송되는 상위 프로토콜

## 출처
- [kurose-ch4](../sources/kurose-ch4.md)
