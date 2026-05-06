# 위키 인덱스

## 계층 (Layers)
- [[layers/application]] — 애플리케이션 계층 개관
- [[layers/transport]] — 전송 계층 개관
- [[layers/network]] — 네트워크 계층 개관
- [[layers/link]] — 링크 계층 개관

## 프로토콜 (Protocols)
- [[HTTP]] — 웹의 요청-응답 프로토콜 (HTTP/1.1, HTTP/2, HTTP/3)
- [[SMTP]] — 이메일 전송 프로토콜
- [[DNS]] — 도메인 이름 시스템 (분산 계층 데이터베이스)
- [[TCP]] — 연결 지향 신뢰적 전송 프로토콜
- [[UDP]] — 비연결형 최소 기능 전송 프로토콜
- [[QUIC]] — UDP 기반 차세대 전송 프로토콜 (HTTP/3 기반)
- [[IPv4]] — 인터넷 핵심 네트워크 프로토콜 (32비트 주소)
- [[IPv6]] — 차세대 IP (128비트 주소, 40바이트 고정 헤더)
- [[DHCP]] — 호스트 IP 주소 자동 할당 프로토콜
- [[OSPF]] — Intra-AS 라우팅 프로토콜 (LS/Dijkstra 기반)
- [[BGP]] — Inter-AS 라우팅 프로토콜 (인터넷의 접착제)
- [[ICMP]] — 인터넷 제어 메시지 프로토콜 (오류 보고, ping, traceroute)
- [[Ethernet]] — 유선 LAN의 지배적 링크 계층 프로토콜
- [[ARP]] — IP 주소를 MAC 주소로 변환 (Address Resolution Protocol)
- [[wifi-80211]] — IEEE 802.11 무선 LAN (WiFi, CSMA/CA)
- [[bluetooth]] — Bluetooth 개인 영역 무선 네트워크 (piconet, FHSS)
- [[lte]] — 4G LTE 셀룰러 네트워크 (all-IP, eNode-B, GTP)
- [[TLS]] — TCP 위 보안 프로토콜 (기밀성, 무결성, 서버 인증)
- [[IPsec]] — 네트워크 계층 보안 프로토콜 (VPN, ESP, SA)
- [[PGP]] — 이메일 보안 프로토콜 (기밀성, 디지털 서명, Web of Trust)

## 개념 (Concepts)
- [[packet-switching]] — 패킷 교환 메커니즘
- [[circuit-switching]] — 회선 교환 메커니즘
- [[protocol-layers]] — 프로토콜 계층 구조 (인터넷 5계층 스택)
- [[encapsulation]] — 캡슐화와 역캡슐화
- [[delay-loss-throughput]] — 지연, 손실, 처리량
- [[internet-structure]] — 인터넷 구조 (ISP 계층, 피어링, IXP)
- [[cdn]] — 콘텐츠 분배 네트워크 (CDN)
- [[multiplexing-demultiplexing]] — 전송 계층 다중화/역다중화
- [[reliable-data-transfer]] — 신뢰적 데이터 전송 원리 (ARQ, GBN, SR)
- [[congestion-control]] — 혼잡 제어 메커니즘 (AIMD, slow start, CUBIC, BBR)
- [[flow-control]] — 흐름 제어 (수신 윈도우)
- [[end-to-end-principle]] — 종단간 설계 원칙
- [[forwarding-routing]] — 포워딩 vs 라우팅, data/control plane
- [[router-architecture]] — 라우터 내부 구조 (스위칭 패브릭, 큐잉, HOL 블로킹)
- [[ip-addressing]] — IP 주소 체계 (서브넷, CIDR, 최장 프리픽스 매칭)
- [[nat]] — 네트워크 주소 변환 (NAT)
- [[packet-scheduling]] — 패킷 스케줄링 (FIFO, Priority, RR, WFQ)
- [[generalized-forwarding]] — 일반화된 포워딩, SDN, OpenFlow
- [[middleboxes]] — 미들박스, NFV, IP 모래시계
- [[routing-algorithms]] — 라우팅 알고리즘 (LS/Dijkstra, DV/Bellman-Ford)
- [[sdn-control-plane]] — SDN 컨트롤 플레인 (컨트롤러, OpenFlow, OpenDaylight/ONOS)
- [[network-management]] — 네트워크 관리 (SNMP/MIB, NETCONF/YANG)
- [[mac-address]] — MAC 주소 (6바이트, 평면적 주소 체계)
- [[error-detection-correction]] — 오류 검출/정정 (패리티, 체크섬, CRC)
- [[multiple-access]] — 다중 접속 프로토콜 (ALOHA, CSMA, CSMA/CD, 순번)
- [[link-layer-switching]] — 링크 계층 스위칭 (self-learning, filtering/forwarding)
- [[vlan]] — VLAN (가상 LAN, 802.1Q 트렁킹)
- [[mpls]] — MPLS (라벨 스위칭, 트래픽 엔지니어링)
- [[data-center-networking]] — 데이터 센터 네트워킹 (Clos, 로드 밸런싱)
- [[wireless-link-characteristics]] — 무선 링크 특성 (경로 손실, SNR/BER, hidden terminal, fading)
- [[cdma]] — 코드 분할 다중 접속 (CDMA, chipping rate, 직교 코드)
- [[cellular-network-architecture]] — 셀룰러 네트워크 아키텍처 (4G/5G, 셀, 기지국, 코어)
- [[mobility-management]] — 이동성 관리 (home/visited network, indirect/direct routing, handover, Mobile IP)
- [[cryptography]] — 암호화 원리 (대칭키/공개키, 블록 암호, CBC, RSA)
- [[message-integrity]] — 메시지 무결성 (해시, MAC/HMAC, 디지털 서명, CA/인증서)
- [[end-point-authentication]] — 종단점 인증 (ap1.0~4.0, nonce, 재생 공격 방지)
- [[firewall]] — 방화벽 (패킷 필터, 상태 기반, 애플리케이션 게이트웨이)
- [[ids-ips]] — 침입 탐지/방지 시스템 (시그니처 기반, 이상 기반, Snort)

## 비교 (Comparisons)

## 소스 (Sources)
- [[kurose-ch1]] — Kurose 8판 1장: Computer Networks and the Internet
- [[kurose-ch2]] — Kurose 8판 2장: Application Layer
- [[kurose-ch3]] — Kurose 8판 3장: Transport Layer
- [[kurose-ch4]] — Kurose 8판 4장: The Network Layer — Data Plane
- [[kurose-ch5]] — Kurose 8판 5장: The Network Layer — Control Plane
- [[kurose-ch6]] — Kurose 8판 6장: The Link Layer and LANs
- [[kurose-ch7]] — Kurose 8판 7장: Wireless and Mobile Networks
- [[kurose-ch8]] — Kurose 8판 8장: Security in Computer Networks
