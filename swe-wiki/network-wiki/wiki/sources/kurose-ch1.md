---
tags: [source, overview]
source: "Kurose & Ross, Computer Networking: A Top-Down Approach, 8th Edition"
chapter: 1
title: "Computer Networks and the Internet"
pages: "31-94"
last_updated: 2026-04-08
---

# Kurose 8판 — Chapter 1: Computer Networks and the Internet

## 핵심 내용

컴퓨터 네트워크와 인터넷의 전체 개관을 다루는 도입 챕터. 인터넷의 구성 요소, 네트워크 코어의 동작 원리, 성능 지표, 프로토콜 계층 구조, 보안 위협, 그리고 네트워킹의 역사를 소개한다.

## 다룬 토픽

### 1.1 인터넷이란?
- **하드웨어/소프트웨어 관점**: 호스트(종단 시스템), 통신 링크, 패킷 스위치(라우터, 링크 계층 스위치), ISP
- **서비스 관점**: 분산 애플리케이션에 서비스를 제공하는 인프라, 소켓 인터페이스
- **프로토콜 정의**: 둘 이상의 통신 엔티티 간 메시지의 형식, 순서, 그리고 메시지 송수신 시 취하는 행동을 정의

### 1.2 네트워크 엣지
- **종단 시스템**: 클라이언트와 서버, 데이터 센터
- **액세스 네트워크**: DSL, 케이블(HFC), FTTH, 5G 고정 무선, 이더넷, WiFi, 3G/4G/5G 셀룰러
- **물리 매체**: 꼬임 쌍선(UTP), 동축 케이블, 광섬유, 지상파 라디오, 위성 라디오

### 1.3 네트워크 코어
- **[[packet-switching|패킷 교환(Packet Switching)]]**: 저장 후 전달(store-and-forward), 큐잉 지연과 패킷 손실, 포워딩 테이블과 라우팅 프로토콜
- **[[circuit-switching|회선 교환(Circuit Switching)]]**: FDM과 TDM, 전용 자원 예약
- **패킷 교환 vs 회선 교환**: 패킷 교환이 통계적 다중화로 더 효율적 (3배 이상의 사용자 지원 가능)
- **[[internet-structure|네트워크의 네트워크]]**: ISP 계층 구조 (access ISP → regional ISP → tier-1 ISP), IXP, 피어링, 콘텐츠 제공자 네트워크

### 1.4 [[delay-loss-throughput|지연, 손실, 처리량]]
- **노드 지연 4요소**: 처리 지연(d_proc), 큐잉 지연(d_queue), 전송 지연(d_trans = L/R), 전파 지연(d_prop = d/s)
- **트래픽 강도(Traffic Intensity)**: La/R — 1에 가까워지면 큐잉 지연 급증
- **종단 간 지연**: d_end-end = N(d_proc + d_trans + d_prop)
- **Traceroute**: 경로 상의 라우터와 RTT 측정
- **처리량(Throughput)**: 병목 링크(bottleneck link)에 의해 결정, min{R_s, R_c}

### 1.5 [[protocol-layers|프로토콜 계층]]과 서비스 모델
- **인터넷 프로토콜 스택 (5계층)**:
  1. [[layers/application|Application]] — 메시지 ([[HTTP]], [[SMTP]], [[DNS]])
  2. [[layers/transport|Transport]] — 세그먼트 ([[TCP]], [[UDP]])
  3. [[layers/network|Network]] — 데이터그램 (IP, 라우팅 프로토콜)
  4. [[layers/link|Link]] — 프레임 ([[Ethernet|이더넷]], WiFi)
  5. Physical — 비트
- **[[encapsulation|캡슐화(Encapsulation)]]**: 각 계층이 헤더를 추가하며 하위 계층으로 전달

### 1.6 네트워크 보안 위협
- 멀웨어(바이러스, 웜), 봇넷
- DoS/DDoS 공격 (취약점 공격, 대역폭 플러딩, 연결 플러딩)
- 패킷 스니핑
- IP 스푸핑

### 1.7 네트워킹의 역사
- 1961-1972: 패킷 교환 개발 (Kleinrock, Baran, Davies), ARPAnet
- 1972-1980: 인터네트워킹, TCP/IP 탄생 (Cerf, Kahn)
- 1980-1990: 네트워크 확산, NSFNET, DNS 개발
- 1990s: WWW 등장, 인터넷 상업화
- 2000s~: 브로드밴드, 모바일, 소셜 네트워크, 클라우드 컴퓨팅

## 위키에 반영된 페이지

- [[packet-switching]] — 패킷 교환 개념
- [[circuit-switching]] — 회선 교환 개념
- [[protocol-layers]] — 프로토콜 계층 구조
- [[encapsulation]] — 캡슐화
- [[delay-loss-throughput]] — 지연, 손실, 처리량
- [[internet-structure]] — 인터넷 구조 (ISP 계층)

## 출처

Kurose, J. F. & Ross, K. W. (2022). *Computer Networking: A Top-Down Approach* (8th Global Edition). Pearson.
