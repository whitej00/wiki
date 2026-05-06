---
tags: [concept, network]
layer: network
related: [lte, cellular-network-architecture, wifi-80211, nat, forwarding-routing]
source_count: 1
last_updated: 2026-04-09
---

# 이동성 관리 (Mobility Management)

이동성 관리는 모바일 장치가 네트워크 접속점(AP 또는 기지국)을 변경하면서도 **진행 중인 통신(TCP 연결 등)을 유지**할 수 있게 하는 네트워크 계층의 메커니즘이다.

## 이동성 스펙트럼

네트워크 관점에서 이동성의 수준은 다양하다:

| 수준 | 설명 | 필요 메커니즘 |
|------|------|-------------|
| (a) 이동 중 전원 끔 | 네트워크 간 이동 시 장치 off → on | 일반 접속 (DHCP 등) |
| (b) 동일 AP 내 이동 | 물리적 이동, 같은 기지국/AP | 없음 |
| (c) 단일 제공자 내 이동 | AP/기지국 변경, 연결 유지 | **핸드오버** |
| (d) 제공자 간 이동 | 다른 캐리어 네트워크로 로밍 | **핸드오버 + 로밍 협정** |

네트워크 계층에서 진정한 이동성은 **(c)**부터 시작한다.

## 핵심 개념

### Home Network / Visited Network

- **홈 네트워크(Home Network)**: 가입자의 영구적 소속 네트워크. **HSS**(Home Subscriber Service)에 가입자 정보(IMSI, 인증 키, 서비스 정보) 저장
- **방문 네트워크(Visited Network)**: 장치가 현재 접속해 있는 타 네트워크. 홈 네트워크가 아니면 **로밍(roaming)** 상태
- 인터넷에서는 홈/방문 네트워크 개념이 약하다 (학교 네트워크, 회사 네트워크 등이 비공식적으로 해당). **Mobile IP**가 이를 공식화하려 했지만 보급은 제한적

### 모바일 장치 식별자

- 4G/5G: **IMSI** (SIM 카드, 글로벌 고유) + NAT IP 주소 (방문 네트워크)
- Mobile IP: **영구 IP 주소** (홈 네트워크) + **care-of-address** (방문 네트워크)

## 라우팅 방식

### Indirect Routing (간접 라우팅)

```
Correspondent → Home Network Gateway → (터널) → Visited Network Gateway → Mobile Device
```

1. Correspondent가 모바일 장치의 영구 주소로 데이터그램 전송
2. 홈 게이트웨이가 HSS 조회 → 방문 네트워크 확인
3. 홈 게이트웨이가 원본 데이터그램을 **캡슐화(encapsulate)**하여 방문 네트워크 게이트웨이로 터널링
4. 방문 게이트웨이가 **역캡슐화(decapsulate)** → NAT 변환 → 모바일 장치에 전달

**모바일 → Correspondent 방향**: 두 가지 옵션
- **(4a)** 홈 게이트웨이를 경유하여 역터널링 (대칭 간접 라우팅, 4G LTE 기본)
- **(4b)** 방문 네트워크에서 직접 전송 (**local breakout** — 표준에 정의되었으나 실제 사용은 제한적)

**장점**: Correspondent에게 이동성이 완전히 투명 (주소 변경 불필요)
**단점**: **삼각 라우팅(Triangle Routing)** — 같은 방문 네트워크에 있는 두 장치가 홈 네트워크를 경유해야 함

### Direct Routing (직접 라우팅)

```
Correspondent → HSS 조회 → Visited Network Gateway → Mobile Device
```

1. Correspondent가 HSS에 모바일 장치의 방문 네트워크 정보 요청
2. HSS가 방문 네트워크 위치 반환
3. Correspondent가 방문 네트워크 게이트웨이로 직접 터널링

**장점**: 삼각 라우팅 해소
**단점**: (1) Correspondent에 모바일 위치 조회 프로토콜 필요 (2) 장치가 다른 방문 네트워크로 이동 시 Correspondent에게 새 위치 알려야 함

## 필요 프로토콜

간접 라우팅을 위해 세 가지 프로토콜이 필요:

1. **모바일 장치 ↔ 방문 네트워크 연관 프로토콜**: 방문 네트워크에 등록/해제
2. **방문 네트워크 → 홈 네트워크 HSS 등록 프로토콜**: 모바일 장치 위치 갱신
3. **홈 게이트웨이 ↔ 방문 게이트웨이 터널링 프로토콜**: 데이터그램 캡슐화/역캡슐화, NAT 변환

## 4G/5G Mobility in Practice

[[lte]] 참조. 실제 4G/5G 이동성은 4단계로 구성:

### 1. Base Station Association
모바일 장치가 방문 네트워크의 기지국과 연관. 모든 주파수에서 동기 신호 탐색 → 기지국 선택 → IMSI 제공.

### 2. Control-Plane Configuration
MME가 IMSI로 HSS 조회 → 상호 인증 → HSS에 장치 위치 등록 → 데이터 채널 파라미터 설정.

### 3. Data-Plane Tunnel Setup
MME가 기지국↔S-GW, S-GW↔P-GW 두 GTP 터널 설정. 이후 모바일 장치가 인터넷 통신 가능.

### 4. Handover (핸드오버)

모바일 장치가 한 기지국에서 다른 기지국으로 접속점을 변경하는 과정:

| 단계 | 내용 |
|------|------|
| 1 | Source BS가 target BS를 선택, Handover Request 전송 |
| 2 | Target BS가 자원 가용 여부 확인, 채널/타임슬롯 사전 할당, Handover Request ACK 반환 |
| 3 | Source BS가 모바일 장치에 target BS 정보 전달 → 모바일 장치가 target BS로 전환 |
| 4 | Source BS가 모바일 장치 행 데이터그램을 target BS로 포워딩 |
| 5 | Target BS가 MME에 통보 → MME가 S-GW에 터널 종단점 변경 지시 |
| 6 | Target BS가 source BS에 자원 해제 확인 |
| 7 | Target BS가 데이터그램 전달 시작 (포워딩된 것 + 새로 도착한 것) |

핸드오버 발생 이유: 신호 열화, 셀 과부하, 부하 분산 등. 기지국이 주기적으로 측정하는 beacon 신호 강도에 기반하여 결정한다. 5G에서는 소형 셀로 인해 핸드오버 빈도가 증가하며 저지연이 더욱 중요해진다.

## Mobile IP (RFC 5944)

인터넷을 위한 이동성 관리 표준. 4G/5G와 매우 유사한 아키텍처:

| 4G/5G 요소 | Mobile IP 요소 | 비고 |
|-----------|-------------|------|
| Home network | Home network | 동일 |
| Visited network | Foreign network | 명칭 차이 |
| IMSI | Permanent IP address | 글로벌 고유 라우팅 가능 주소 |
| HSS | Home agent | 위치 추적 |
| MME | Foreign agent | 이동성 관리 |
| GTP 터널 (간접 라우팅) | 간접 라우팅 + 터널링 | 동일 원리 |

Mobile IP의 세 구성요소:
1. **Agent discovery**: foreign agent가 이동성 서비스를 광고, care-of-address 제공
2. **Home agent 등록**: 모바일 장치가 care-of-address를 home agent에 등록/해제
3. **간접 라우팅**: home agent가 데이터그램을 foreign agent로 터널링

보급이 제한적인 이유: 셀룰러 네트워크가 이미 이동성 솔루션을 제공하고 있어 별도의 인터넷 계층 이동성 프로토콜의 필요성이 낮음.

## 무선과 이동성이 상위 계층에 미치는 영향

### TCP에 대한 영향

- TCP는 세그먼트 손실을 **혼잡**으로 해석하여 전송률을 줄이지만, 무선에서는 **비트 오류**나 **핸드오버 손실**이 주 원인
- 불필요한 혼잡 윈도우 축소 → 성능 저하

### 대응 방안

| 접근 | 설명 |
|------|------|
| **Local recovery** | 무선 링크에서 ARQ/FEC로 오류 복구 (802.11 link-layer ACK, LTE RLC) |
| **TCP sender awareness** | TCP 송신자가 유선/무선 손실을 구분하여 혼잡 제어 적용 |
| **Split-connection** | 무선 구간과 유선 구간을 별도 전송 계층 연결로 분리 (셀룰러에서 널리 사용) |

### 애플리케이션 계층

- 무선 대역폭은 제한적 → 콘텐츠 적응 필요 (예: 모바일 웹 서버)
- 이동성은 **위치 기반 서비스**(GPS + WiFi positioning)라는 새로운 가능성을 열음

## 출처

- [[kurose-ch7]] — Kurose 8판 7장 (7.5, 7.6, 7.7절)
