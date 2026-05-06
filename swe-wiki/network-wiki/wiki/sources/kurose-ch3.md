---
tags: [source, kurose, transport-layer]
source_type: textbook
chapter: 3
pages: 211-314
last_updated: 2026-04-08
---

# Kurose 8판 Chapter 3: Transport Layer

## 소스 정보
- **교재**: Computer Networking: A Top-Down Approach, 8th Edition
- **저자**: James F. Kurose, Keith W. Ross
- **챕터**: 3 — Transport Layer
- **페이지**: pp. 211–314

## 핵심 내용

### 3.1 [전송 계층](../layers/transport.md) 소개
- 전송 계층은 **논리적 통신(Logical Communication)**을 애플리케이션 프로세스 간에 제공
- 네트워크 계층이 호스트 간 통신을 제공하는 것과 구별됨
- 전송 계층 프로토콜은 종단 시스템(end system)에서만 동작, 중간 라우터는 관여하지 않음
- IP는 **최선형 전달 서비스(Best-effort Delivery Service)** — 신뢰성 보장 없음
- 인터넷의 전송 프로토콜: **[TCP](../protocols/tcp.md)**(신뢰적, 연결 지향)와 **[UDP](../protocols/udp.md)**(비신뢰적, 비연결)

### 3.2 [다중화와 역다중화](../concepts/multiplexing-demultiplexing.md)(Multiplexing & Demultiplexing)
- 호스트-호스트 전달을 프로세스-프로세스 전달로 확장하는 핵심 기능
- **다중화**: 소켓에서 데이터를 모아 세그먼트 생성 후 네트워크 계층으로 전달
- **역다중화**: 수신된 세그먼트를 올바른 소켓으로 전달
- 포트 번호: 16비트(0–65535), 0–1023은 well-known ports
- UDP 소켓 식별: **(목적지 IP, 목적지 포트)** 2-tuple
- TCP 소켓 식별: **(출발지 IP, 출발지 포트, 목적지 IP, 목적지 포트)** 4-tuple

### 3.3 비연결형 전송: [UDP](../protocols/udp.md)
- RFC 768, 최소 기능의 전송 프로토콜 ([다중화/역다중화](../concepts/multiplexing-demultiplexing.md) + 오류 검출)
- 비연결형(Connectionless) — 핸드셰이크 없이 즉시 전송
- UDP 선호 이유: (1) 전송 시점/속도 제어 가능, (2) 연결 설정 지연 없음, (3) 연결 상태 불필요, (4) 작은 헤더 오버헤드(8바이트 vs TCP 20바이트)
- **UDP 세그먼트 구조**: 출발지 포트(16비트), 목적지 포트(16비트), 길이(16비트), 체크섬(16비트)
- **체크섬**: 16비트 워드의 합의 1의 보수. **End-to-end 원칙**의 예시
- UDP 사용 애플리케이션: DNS, SNMP, NFS, 실시간 멀티미디어

### 3.4 [신뢰적 데이터 전송 원리](../concepts/reliable-data-transfer.md)
- 네트워킹의 가장 근본적 문제 중 하나
- **rdt1.0**: 완전 신뢰 채널 — 오류 처리 불필요
- **rdt2.0**: 비트 오류 채널 — **ARQ**(Automatic Repeat reQuest): 체크섬 + ACK/NAK + 재전송
- **rdt2.1**: ACK/NAK 손상 대비 — **순서번호(Sequence Number)** 도입 (0, 1)
- **rdt2.2**: NAK 제거 — 중복 ACK로 대체
- **rdt3.0**: 패킷 손실 채널 — **카운트다운 타이머** 도입. **교대 비트 프로토콜(Alternating-bit Protocol)**
- **파이프라이닝(Pipelining)**: stop-and-wait의 낮은 이용률 해결. 여러 패킷을 ACK 대기 없이 전송
- **Go-Back-N (GBN)**: 윈도우 크기 N, **누적 확인응답(Cumulative ACK)**, 타임아웃 시 윈도우 내 전체 재전송, 수신자는 순서 맞지 않는 패킷 폐기
- **Selective Repeat (SR)**: 개별 확인응답, 손실된 패킷만 재전송, 수신자가 비순서 패킷 버퍼링. 윈도우 크기 ≤ 순서번호 공간/2

### 3.5 연결 지향 전송: [TCP](../protocols/tcp.md)
- RFC 793, RFC 5681 등
- **연결 지향(Connection-oriented)**, **전이중(Full-duplex)**, **점대점(Point-to-point)**
- **MSS**(Maximum Segment Size): 세그먼트 최대 데이터 크기. **MTU**(Maximum Transmission Unit)에서 TCP/IP 헤더(40바이트) 제외. 일반적 MSS = 1460바이트
- **TCP 세그먼트 구조**: 출발지/목적지 포트, 순서번호(32비트), 확인응답번호(32비트), 헤더 길이(4비트), 플래그(URG/ACK/PSH/RST/SYN/FIN + CWR/ECE), 수신 윈도우(16비트), 체크섬, 긴급 데이터 포인터
- **순서번호**: 바이트 스트림 기준 (세그먼트 첫 바이트의 번호)
- **확인응답번호**: 수신 측이 기대하는 다음 바이트 번호. **누적 확인응답(Cumulative Acknowledgment)**
- **RTT 추정**: SampleRTT → EstimatedRTT = 0.875·EstimatedRTT + 0.125·SampleRTT (**EWMA**). DevRTT = 0.75·DevRTT + 0.25·|SampleRTT − EstimatedRTT|. TimeoutInterval = EstimatedRTT + 4·DevRTT
- **신뢰적 전송**: 단일 재전송 타이머, 가장 오래된 미확인 세그먼트 재전송. 타임아웃 시 간격 2배(지수적 백오프)
- **Fast Retransmit**: 3개 중복 ACK 수신 시 타이머 만료 전 재전송
- TCP는 **GBN과 SR의 하이브리드**. Selective Acknowledgment(SACK, RFC 2018) 확장 가능
- **[흐름 제어(Flow Control)](../concepts/flow-control.md)**: 수신 윈도우(rwnd)로 송신자 전송률 제한. rwnd = RcvBuffer − (LastByteRcvd − LastByteRead). LastByteSent − LastByteAcked ≤ rwnd
- **연결 관리**: 3-way handshake (SYN → SYNACK → ACK). 연결 해제: FIN → ACK → FIN → ACK, TIME_WAIT 상태
- **TCP 상태 전이**: CLOSED → SYN_SENT → ESTABLISHED → FIN_WAIT_1 → FIN_WAIT_2 → TIME_WAIT → CLOSED (클라이언트). CLOSED → LISTEN → SYN_RCVD → ESTABLISHED → CLOSE_WAIT → LAST_ACK → CLOSED (서버)
- **SYN Flood 공격**: 대량 SYN 전송으로 서버 자원 고갈. **SYN 쿠키** 방어: 서버가 연결 상태 저장 없이 해시 기반 ISN으로 SYNACK 응답
- **nmap 포트 스캐닝**: TCP SYN 세그먼트로 열린 포트 탐지

### 3.6 혼잡 제어 원리
- **혼잡의 비용**: (1) 큐잉 지연 증가, (2) 패킷 손실로 인한 재전송, (3) 불필요한 재전송으로 대역폭 낭비, (4) 다중 홉에서 드롭 시 업스트림 전송 용량 낭비
- 시나리오 1: 무한 버퍼, 처리량은 R/2에 수렴하나 지연 무한 증가
- 시나리오 2: 유한 버퍼, offered load(λ'_in) 중 재전송 비율 증가
- 시나리오 3: 다중 홉, 처리량이 0으로 감소 가능 (congestion collapse)
- **접근 방식**: End-to-end 혼잡 제어 (TCP) vs Network-assisted 혼잡 제어 (ATM ABR, ECN)

### 3.7 [TCP 혼잡 제어](../concepts/congestion-control.md)
- **혼잡 윈도우(cwnd)**: 미확인 데이터량 제한. 전송률 ≈ cwnd/RTT
- 손실 이벤트 = 타임아웃 또는 3개 중복 ACK
- **Self-clocking**: ACK 도착 속도에 맞춰 윈도우 증가
- **Slow Start**: cwnd = 1 MSS에서 시작, 매 ACK마다 1 MSS 증가 → RTT마다 2배 (지수적 증가). ssthresh 도달 시 congestion avoidance 전환
- **Congestion Avoidance**: RTT마다 cwnd를 1 MSS씩 증가 (선형 증가). 타임아웃 시 cwnd = 1 MSS, ssthresh = cwnd/2
- **Fast Recovery**: 3중복 ACK 시 ssthresh = cwnd/2, cwnd = ssthresh + 3·MSS. 중복 ACK마다 cwnd += 1 MSS. 새 ACK 도착 시 congestion avoidance로 전환
- **TCP Tahoe**: 모든 손실 이벤트에 cwnd = 1 MSS로 리셋
- **TCP Reno**: 3중복 ACK에는 fast recovery, 타임아웃에만 cwnd = 1 MSS
- **AIMD**(Additive Increase, Multiplicative Decrease): 톱니파형(sawtooth) 패턴. 평균 처리량 ≈ 0.75·W/RTT
- **TCP CUBIC**: 큐빅 함수로 cwnd 증가. W_max 근처에서 천천히 탐색, 멀리서 빠르게 증가. Linux 기본 TCP
- **ECN**(Explicit Congestion Notification): IP 헤더 ToS 필드 2비트. 라우터가 혼잡 표시 → 수신자가 ECE 비트 설정 → 송신자가 cwnd 절반, CWR 설정
- **TCP Vegas**: RTT 지연 측정 기반. cwnd/RTT_min과 실측 처리량 비교
- **BBR**: 지연 기반, Google 개발. B4 네트워크, YouTube 등에 배포
- **공정성**: AIMD는 동일 RTT 연결 간 대역폭 균등 분배로 수렴. 그러나 RTT 차이, UDP 트래픽, 병렬 TCP 연결이 공정성 저해
- **TCP Splitting**: 프론트엔드 서버 경유로 slow start 지연 감소 (4·RTT → RTT 수준)

### 3.8 전송 계층 기능의 진화
- TCP/UDP 외 새로운 전송 프로토콜 등장
- TCP 변종: CUBIC, DCTCP, CTCP, BBR 등
- **[QUIC](../protocols/quic.md)**(Quick UDP Internet Connections): [UDP](../protocols/udp.md) 기반 애플리케이션 계층 프로토콜. 연결 지향 + 암호화, 스트림 멀티플렉싱, 스트림별 [신뢰적 전송](../concepts/reliable-data-transfer.md)(HOL 블로킹 해결), TCP NewReno 기반 [혼잡 제어](../concepts/congestion-control.md). HTTP/3의 기반

## 다룬 토픽 목록
- [multiplexing-demultiplexing](../concepts/multiplexing-demultiplexing.md)
- [UDP](../protocols/udp.md)
- [TCP](../protocols/tcp.md)
- [reliable-data-transfer](../concepts/reliable-data-transfer.md)
- [congestion-control](../concepts/congestion-control.md)
- [flow-control](../concepts/flow-control.md)
- [end-to-end-principle](../concepts/end-to-end-principle.md)
- [QUIC](../protocols/quic.md)
- [transport](../layers/transport.md)

## 위키에 반영된 페이지
- protocols/tcp.md (신규)
- protocols/udp.md (신규)
- protocols/quic.md (신규)
- concepts/multiplexing-demultiplexing.md (신규)
- concepts/reliable-data-transfer.md (신규)
- concepts/congestion-control.md (신규)
- concepts/flow-control.md (신규)
- concepts/end-to-end-principle.md (신규)
- layers/transport.md (신규)
