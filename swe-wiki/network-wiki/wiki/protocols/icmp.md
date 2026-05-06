---
tags: [protocol, network-layer]
layer: network
rfc: 792
related: [IPv4, IPv6, ip-addressing]
source_count: 1
last_updated: 2026-04-08
---

# ICMP (Internet Control Message Protocol)

호스트와 라우터가 네트워크 계층 정보를 주고받기 위한 프로토콜. 주로 **오류 보고**에 사용되며, ping과 traceroute의 기반이 된다.

## 한 줄 요약

IP 데이터그램 위에서 동작하는 네트워크 진단·오류 보고 프로토콜.

## 프로토콜 위치

ICMP는 구조적으로 IP 바로 위에 위치한다. ICMP 메시지는 **IP 데이터그램의 페이로드**로 운반되며, IP 헤더의 upper-layer protocol 번호 **1**로 식별된다. TCP/UDP 세그먼트와 동일한 방식으로 캡슐화된다.

## 메시지 형식

ICMP 메시지는 **type**과 **code** 필드를 가지며, 오류를 유발한 IP 데이터그램의 **헤더 + 첫 8바이트**를 포함한다 (송신자가 원인 데이터그램을 식별할 수 있도록).

## 주요 메시지 타입

| Type | Code | 설명 |
|------|------|------|
| 0 | 0 | Echo reply (ping 응답) |
| 3 | 0 | Destination network unreachable |
| 3 | 1 | Destination host unreachable |
| 3 | 2 | Destination protocol unreachable |
| 3 | 3 | Destination port unreachable |
| 3 | 6 | Destination network unknown |
| 3 | 7 | Destination host unknown |
| 4 | 0 | Source quench (혼잡 제어, 현재 미사용) |
| 8 | 0 | Echo request (ping 요청) |
| 9 | 0 | Router advertisement |
| 10 | 0 | Router discovery |
| 11 | 0 | TTL expired (traceroute 핵심) |
| 12 | 0 | IP header bad |

## 주요 활용

### ping
- 송신자가 **type 8 code 0** (echo request) 전송
- 목적지가 **type 0 code 0** (echo reply) 응답
- ping 서버는 OS 커널에서 직접 처리 (별도 프로세스 아님)
- 클라이언트는 OS에 ICMP 메시지 생성을 요청해야 함

### traceroute
경로상의 라우터를 추적하는 프로그램:

1. TTL을 **1부터 증가**시키며 UDP 세그먼트(사용하지 않을 법한 포트 번호)를 목적지로 전송
2. TTL이 0이 되는 n번째 라우터가 데이터그램을 폐기하고 **type 11 code 0** (TTL expired) ICMP를 송신자에게 반환
3. 송신자는 ICMP 메시지에서 n번째 라우터의 이름과 IP, 그리고 타이머에서 RTT를 얻음
4. TTL이 충분히 커서 목적지에 도달하면, 목적지가 존재하지 않는 포트에 대해 **type 3 code 3** (port unreachable) 반환 → traceroute 종료
5. 표준 traceroute는 **각 TTL 값에 대해 3개 패킷** 전송 (3개 RTT 결과)

## ICMPv6

IPv6용 ICMP [RFC 4443]:
- 기존 type/code 재정의
- 새로운 기능 추가: **Packet Too Big** (IPv6에서는 라우터가 단편화하지 않으므로 필수), "unrecognized IPv6 options" 오류 코드

## Source Quench (Type 4)

원래 혼잡 제어를 위해 설계됨 — 혼잡한 라우터가 송신 호스트에게 전송률 감소를 요청. 그러나 TCP가 자체 [혼잡 제어](../concepts/congestion-control.md) 메커니즘을 가지고 있고, [ECN](ipv4.md) 비트가 혼잡 시그널링에 사용되면서 현재는 거의 사용되지 않는다.

## 출처
- [kurose-ch5](../sources/kurose-ch5.md)
