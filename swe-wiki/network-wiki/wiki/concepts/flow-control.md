---
tags: [concept, transport-layer]
layer: transport
related: [TCP, UDP, congestion-control]
source_count: 1
last_updated: 2026-04-08
---

# 흐름 제어 (Flow Control)

[TCP](../protocols/tcp.md)가 제공하는 **속도 매칭 서비스(Speed-matching Service)**로, 송신자가 수신자의 버퍼를 오버플로시키지 않도록 전송률을 조절하는 메커니즘이다.

## 왜 필요한지

TCP 연결의 수신 측은 **수신 버퍼(Receive Buffer)**를 가진다. 정상 수신된 데이터는 이 버퍼에 저장되고, 애플리케이션 프로세스가 읽어간다. 그러나 애플리케이션이 데이터를 즉시 읽지 않을 수 있고, 다른 작업에 바쁘거나 읽기 속도가 느릴 수 있다. 송신자가 수신자보다 빠르게 보내면 **버퍼 오버플로**로 데이터 손실이 발생한다.

## 흐름 제어 vs 혼잡 제어

두 메커니즘은 모두 송신자의 전송률을 조절하지만 **원인이 다르다**:

| | 흐름 제어 | [혼잡 제어](congestion-control.md) |
|---|---------|------------|
| **보호 대상** | 수신자 버퍼 | 네트워크 |
| **원인** | 수신 애플리케이션의 느린 읽기 | 네트워크 혼잡 (라우터 버퍼 오버플로) |
| **피드백** | 수신 윈도우(rwnd) | 패킷 손실, RTT 증가, ECN |

많은 저자가 두 용어를 혼용하지만, 구별하는 것이 중요하다.

## 수신 윈도우 (Receive Window, rwnd)

TCP는 송신자에게 **수신 윈도우(rwnd)** 변수를 유지하도록 하여 흐름 제어를 구현한다.

### 수신 측 변수

- **RcvBuffer**: 수신 버퍼의 총 크기
- **LastByteRcvd**: 네트워크에서 도착하여 버퍼에 저장된 마지막 바이트 번호
- **LastByteRead**: 애플리케이션이 버퍼에서 읽은 마지막 바이트 번호

버퍼 오버플로 방지 조건:
```
LastByteRcvd − LastByteRead ≤ RcvBuffer
```

수신 윈도우:
```
rwnd = RcvBuffer − (LastByteRcvd − LastByteRead)
```

rwnd는 버퍼의 **여유 공간**을 나타내며, 시간에 따라 동적으로 변한다.

### 송신 측 제약

수신자는 매 세그먼트의 **수신 윈도우 필드(16비트)**에 현재 rwnd 값을 넣어 송신자에게 알린다.

송신자는 다음 조건을 유지:
```
LastByteSent − LastByteAcked ≤ rwnd
```

즉, **미확인 데이터량(unacknowledged data)이 rwnd를 초과하지 않도록** 한다.

초기값: rwnd = RcvBuffer

### rwnd = 0 문제와 해결

수신 버퍼가 가득 차면 rwnd = 0을 광고한다. 이 상태에서:
- 수신 애플리케이션이 버퍼를 비워도, 보낼 데이터나 ACK할 세그먼트가 없으면 TCP는 새 rwnd 값을 송신자에게 알릴 수 없음
- 송신자는 영원히 차단(blocked) 상태에 빠질 수 있음

**해결**: TCP 명세는 rwnd = 0일 때 송신자가 **1바이트 데이터를 담은 세그먼트를 계속 전송**하도록 요구한다. 이 세그먼트에 대한 ACK에 새로운 rwnd 값이 포함되어 차단이 해소된다.

## UDP와 흐름 제어

[UDP](../protocols/udp.md)는 흐름 제어를 **제공하지 않는다**. 수신 측 소켓에 유한 크기 버퍼가 있으며, 애플리케이션이 충분히 빠르게 읽지 않으면 세그먼트가 **폐기**된다.

## 출처
- [kurose-ch3](../sources/kurose-ch3.md)
